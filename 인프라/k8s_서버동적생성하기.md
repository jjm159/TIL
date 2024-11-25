# 쿠버네티스로 서버 동적 실행하기
- 쿠버네티스(Kubernetes)와 Spring Boot를 활용하면 게임이 시작될 때마다 새로운 서버를 동적으로 생성하여 실행하는 시스템 구축하기

## 질문
- 만약에 게임이 시작되면, 해당 게임 한 판을 실행하기 위한 서버를 새로 띄우고 싶음
- 쿠버네티스랑 springboot를 이용해서 구현이 가능할까?
- 동적으로 생성되는 새로운 서버의 ip나 port 같은 관리를 어떻게 할 수 있을까?

## 구현 가능성
- 쿠버네티스는 동적으로 Pod(컨테이너)와 서비스를 생성하고 관리하는 데 최적화된 플랫폼
- 가능한 작업들
    1. 게임 한 판을 위한 서버 Pod 동적 생성
        - Spring Boot 애플리케이션을 `컨테이너화`하여 Pod으로 실행
    2. Pod의 IP 및 Port 관리
        - 쿠버네티스의 서비스(Service) 및 DNS를 사용하여 `동적으로 생성된 Pod`의 `IP` 및 `Port`를 관리
    3. 게임 매니저 서비스
        - Spring Boot 기반 게임 매니저 애플리케이션이 새로운 게임 `서버를 동적으로 생성`하고 관리

---

## 2. 설계 구조

#### 2.1 주요 구성 요소
- Spring Boot 기반 `게임 매니저`
  - 게임 시작 요청을 받고, `새로운 게임 서버`를 생성
  - 생성된 게임 서버의 `IP와 Port 정보`를 클라이언트에게 제공
- 게임 서버(Spring Boot 애플리케이션)
  - 한 판의 게임을 실행
  - 게임이 종료되면 Pod을 삭제
- 쿠버네티스 오브젝트
  - Deployment
    - 게임 서버 Pod의 템플릿 관리
  - Service
    - 동적으로 생성된 Pod에 클라이언트가 접근할 수 있도록 Load Balancing

#### 2.2 동작 흐름
- 클라이언트가 게임 매니저(Spring Boot API)에 게임 시작 요청을 보냄
- 게임 매니저가 쿠버네티스 API를 호출하여 새로운 게임 서버 Pod 생성 요청
- 쿠버네티스가 Pod을 생성하고, Pod에 접근하기 위한 Service를 동적으로 생성
- 게임 매니저가 생성된 게임 서버의 IP와 Port 정보를 클라이언트에 반환
- 클라이언트가 해당 IP와 Port로 게임 서버에 연결

---

## 3. 구현 단계

#### 3.1 게임 서버 이미지 빌드
- Spring Boot로 게임 서버를 개발하고 Docker 이미지로 빌드
- Dockerfile
```Dockerfile
FROM openjdk:17-jdk-slim
COPY target/game-server.jar /app/game-server.jar
ENTRYPOINT ["java", "-jar", "/app/game-server.jar"]
```
- Docker 이미지 빌드
```shell
$ docker build -t my-game-server .
```

#### 3.2 쿠버네티스 리소스 정의
- Deployment 템플릿 (game-server-deployment.yaml)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: game-server
spec:
  replicas: 0 # 시작 시 Pod이 생성되지 않음
  selector:
    matchLabels:
      app: game-server
  template:
    metadata:
      labels:
        app: game-server
    spec:
      containers:
      - name: game-server
        image: my-game-server:latest
        ports:
        - containerPort: 8080 # 게임 서버 포트
```

- Service 템플릿 (game-server-service.yaml)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: game-server
spec:
  selector:
    app: game-server
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: NodePort # 클라이언트가 외부에서 접근 가능
```

#### 3.3 Spring Boot 게임 매니저
- `Kubernetes API`를 활용하여 동적 생성
- Spring Boot에서 쿠버네티스 API를 호출하여 Pod 및 Service를 생성
- KubernetesClient를 활용한 동적 생성
```java
import io.fabric8.kubernetes.client.KubernetesClient;
import io.fabric8.kubernetes.client.dsl.PodResource;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class GameManagerService {

    @Autowired
    private KubernetesClient kubernetesClient;

    public String startGame() {
        // Pod 이름 생성
        String podName = "game-server-" + System.currentTimeMillis();

        // Pod 생성
        kubernetesClient.pods().createNew()
            .withNewMetadata()
                .withName(podName)
                .addToLabels("app", "game-server")
            .endMetadata()
            .withNewSpec()
                .addNewContainer()
                    .withName("game-server")
                    .withImage("my-game-server:latest")
                    .addNewPort()
                        .withContainerPort(8080)
                    .endPort()
                .endContainer()
            .endSpec()
            .done();

        // Service 생성
        kubernetesClient.services().createNew()
            .withNewMetadata()
                .withName(podName + "-service")
            .endMetadata()
            .withNewSpec()
                .addToSelector("app", "game-server")
                .addNewPort()
                    .withPort(8080)
                    .withTargetPort(new IntOrString(8080))
                .endPort()
                .withType("NodePort")
            .endSpec()
            .done();

        // 동적으로 생성된 서비스 IP 및 Port 반환
        return kubernetesClient.services().withName(podName + "-service").getURL();
    }
}
```

---

#### 3.4 게임 종료 및 리소스 정리
- 게임 종료 시 Pod과 Service를 삭제
```java
public void endGame(String podName) {
    kubernetesClient.pods().withName(podName).delete();
    kubernetesClient.services().withName(podName + "-service").delete();
}
```

---

## 4. IP 및 Port 관리
- 쿠버네티스 `DNS` 활용
- 쿠버네티스는 동적으로 생성된 Pod과 Service에 대해 DNS 이름을 제공, 이를 통해 IP와 Port를 간단히 관리
  - 서비스 이름
    - <service-name>.<namespace>.svc.cluster.local
  - 예
    - game-server-service.default.svc.cluster.local:8080

---

## 5. 정리
1. 쿠버네티스
    - Pod 및 Service를 동적으로 생성하여 게임 서버 실행
    - DNS를 사용해 IP 및 Port를 관리
2. Spring Boot
    - 게임 매니저에서 쿠버네티스 API를 호출해 서버 관리
    - 생성된 서버의 정보를 클라이언트에게 제공
3. 실제 사용
    - 게임 시작 시 서버가 자동으로 생성되고, 게임 종료 시 리소스가 정리

## 참고
- [k8s api java client](https://github.com/jjm159/TIL/blob/main/%EC%9D%B8%ED%94%84%EB%9D%BC/k8s_%EC%84%9C%EB%B2%84%EB%8F%99%EC%A0%81%EC%83%9D%EC%84%B1%ED%95%98%EA%B8%B0.md)