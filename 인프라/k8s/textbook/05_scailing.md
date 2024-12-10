# Scailing
- 스케일링? -> 파드를 늘린다!
- __`레플리카(replica)`__
    - 동일한 애플리케이션이 돌아가는 파드
    - 레플리카는 여러 노드에 분산 배치 된다.
        - 고가용성 이점을 누릴 수 있다.
- 여러 방식
    - 디플로이먼트
    - 다른 리소스도 있음

## 1. 쿠버네티스는 어떻게 애플리케이션을 `스케일링`하는가
- 컨트롤러 리소스는 파드를 생성하고 대체하기 위해 템플릿 사용
- 템플릿으로 똑같은 파드의 레플리카를 여러 개 만들 수 있다.
- 디플로이가 직접 파드를 관리하지 않고, `레플리카셋(ReplicaSet)`이 관리한다.
- 정리
    - __`디플로이`는 `레플리카셋`을 관리하는 컨트롤러 리소스__
    - __`레플리카셋`은 `파드`를 관리하는 컨트롤러 리소스__
- 레플리카셋 정의 (디플로이 없이 가능)
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
    name: whoami-web
spec:
    replicas: 1
    selector:
        matchLabels:
            app: whoami-web # 이 레이블을 가진 파드들과 매치됨
    template: # 파드의 정의가 이어짐
```
- 제어 루프
    - 레플리카셋은 제어루프를 돌면서 pod의 수가 관리 중인 리소스 수와 맞는지 확인
    - 모자라면 추가
    - 많으면 삭제
- 의문 1: 예제에서 스케일링이 빠른 이유
    - 단일 노드이기 때문
    - 노드에 이미지가 이미 존재
    - 해당 이미지를 사용해서 pod를 추가하면 끝
    - 만약 처음 컨테이너를 띄우는 거라면, 해당 노드에서 이미지를 내려 받는 시간이 걸리게 된다.
- 의문 2: HTTP 응답이 다른 파드에서 온 이유
    - 느슨한 결합 - 서비스와 파드 간
    - 서비스의 레이블과 일치하는, 늘어난 파드들의 요청도 고르게 분배한다.
    - 참고로 로드밸런싱은 모든 서비스 유형이 가진 기능이다. 
        - 꼭 로드밸런서 타입이 아니어도, 서비스의 레이블과 일치하는 여러 파드에 로드밸런싱 기능이 적용된다.

## 2. `디플로이먼트`와 `레플리카셋`을 이용한 부하 스케일링
- 디플로이먼트는 어따 써?
    - 업그레이드, 롤백 등 추가 기능들이 있음
    - 기본적으로 `배치` 책임을 가짐
- 디플로이먼트의 통제권
    - 관리 주체 - 관리 대상 관계
    - 디플로이먼트를 스케일링하면 레플리카셋의 레플리카 수만 변경하면 끝
    - 파드 정의 변경 시, 대체 레플리카셋을 생성하고, 기존 레플리카셋의 레플리카 수를 0으로 만든다.
    - 새로운 레플리카셋이 완전히 준비가 되면, 그 때 기존의 레플리카셋의 레플리카 수가 줄어든다.
    - 이렇게 스케일링은 레플리카셋이 하고, 레플리카의 배치는 디플로이먼트가 하면서, 스케일링에 대한 우연한 통제권을 가지게 되었다.
    - 이렇게 안했으면, 레플리카셋 교체를 저런식으로 할 수 없었을 것
    - __다시 한번, `스케일링은 레플리카셋`이, `배치는 디플로이먼트`가!! -> 분리!!__
- __repolicaSet을 재활용__
    - replica가 3인 A replicaSet이 적용 중
    - `kubectl scale --replicas=4 app` 으로 4로 변경 요청
    - 새로운 B replicaSet을 생성해서 4로 만들고, 기존 A replicaSet은 0으로 만듬
    - 이 때 A는 바로 사라지지 않음
    - 이 상황에서 다시 3 replica인 메니페스트를 apply
    - 원래 3이었던 A replicaSet을 재활용하고, B를 0으로 만듬
- __재활용이 빠르게 가능한 이유__
    - pod template을 hash로 만들어서 pod의 이름을 짓고, 이걸로 빠르게 찾을 수 있음
    - 새로운 정의를 해시로 만들어서 이미 존재하면 재활용하고 아니면 새로 생성
    - 이렇게 재활용할 수 있기 때문에 기존에 있는 replicaSet을 바로 삭제하지 않고 0으로 만들어서 유지 하고 있는 것
- 서비스의 스케일링
    - 서비스는 로드밸런싱 기능을 사용해서 레플리카셋이 만든 여러 파드에 요청을 분산
    - pod가 스케일링해도 문제 없음
- 스토리지의 스케일링
    - 컨피그맵은 읽기 전용이기 때문에 모든 파드에서 공유 가능
    - 공디렉터리는 pod 단위이기 때문에 파드끼리 공유가 안된다.
    - 이럴 때 사용하는 게 `스테이풀셋`, `데몬셋`!!!

## 3. `데몬셋`을 이용한 스케일링으로 고가용성 확보하기
- __`DaemonSet`__
    - 클러스터 내 모든 노드 또는 셀렉터와 일치하는 일부 노드에서 단일 레플리카 또는 파드로 동작하는 리소스
    - 각 노드에서 정보를 수집, 중앙의 수집 모듈에 전달하거나 인프라 수준의 관심사와 관련된 목적으로 사용
    - 각 노드마다 파드가 하나씩 동작하여 해당 노드의 데이터를 수집하는 방식
- 단순히 고가용성을 확보하기 위해 데몬셋을 활용할 수도 있음
    - 리버스 프록시
    - nginx 하나로도 수천 개의 동시 요청 처리 가능
    - 파드가 여러개일 필요가 없고, 한 노드에 하나씩 배치되는 것이 보장되기만 하면 됨
    - 외부 트래픽이 어떤 노드에 도달하더라도 리버스 프록시가 해당 트래픽을 받아서 전달할 것
    - `프록시 용도로 사용할 데몬셋`
    ```yaml
    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
        name: pi-proxy
    spec:
        selector:
            matchLabels:
                pp: pi-proxy
    template:
        metadata:
            labels:
                app: pi-proxy
        spec:
        # ..
    ```
- __특정 노드에만 데몬셋__ 
    - 노드에 label을 부여하고, 해당 노드에만 실행되도록 하기
    - nodeSelector를 사용한다.
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: pi-proxy
  labels:
    kiamol: ch06
spec:
  selector:
    matchLabels:
      app: pi-proxy
  template:
    metadata:
      labels:
        app: pi-proxy
    spec:
      containers:
        - image: nginx:1.17-alpine
          name: nginx
          ports:
            - containerPort: 80
              name: http
          volumeMounts:
            - name: config
              mountPath: "/etc/nginx/"
              readOnly: true
            - name: cache-volume
              mountPath: /data/nginx/cache
      volumes:
        - name: config
          configMap:
            name: pi-proxy-configmap
        - name: cache-volume
          hostPath:
            path: /volumes/nginx-cache
            type: DirectoryOrCreate
      nodeSelector: # 이거!!!!!!
        kiamol: ch06
```
- 그리고 노드에 label 부여하기
    - `kubectl label node $(kubectl get nodes -o jsonpath='{.items[0].metadata.name}') kiamol=ch06 --overwrite`
- 언제?
    - 각각의 인스턴스가 독립적인 데이터 저장소를 가져도 괜찮은 애플리케이션
    - 여러 인스턴스가 데이터 저장소를 공유해야 하는 경우에는 `StatefulSet` 사용!!
- __애플리케이션 모델링__
    - 스테이트풀셋, 데몬셋, 레플리카셋, 디플로이먼트
    - 모두 애플리케이션을 모델링하는 데 사용하는 도구
    - 원하는 애플리케이션을 실행하기 위해 애플리케이션을 유연하게 모델링할 수 있어야 함!

## 4. 쿠버네티스의 객체 간 `오너십`
- 컨트롤러 리소스를 삭제하면 관리 대상 리소스도 곧 삭제됨
    - `카비지 컬렉터`가 오너가 없어진 객체를 찾아서 삭제해줌
- 객체 간 이런 `오너십`은 일종의 `위계`를 형성
- `파드`는 `레플리카셋`의 관리를 받고, `레플리카셋`은 다시 `디플로이먼트`의 관리를 받는 식
- __이런 관계가 형성되는 수단은 `레이블 셀렉터` 하나 뿐!!__

