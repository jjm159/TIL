# Volume
- k8s에는 클러스터 전체에서 사용 가능한 스토리지를 제공하는 내장기능이 없다.
- 모든 상황에서 적합한 스토리지를 제공할 수 있는 단일 수단은 존재하지 않기 때문
- k8s는 여러 노드를 관리하는데 다양한 노드의 환경마다 스토리지가 다르고,
- 애플리케이션 마다 스토리지에 대한 요구사항이 천차만별!
- `클러스터에서 제공되는 스토리지` 또는 `애플리케이션에 필요한 스토리지 요구 사항을 기술할 수 있는 스토리지 유형`
    - 이 둘을 직접 정의할 수 있도록 해서 문제 해결
- k8s는 스토리지를 어떤 방식으로 추상화 했을까

## 1. k8s에서 컨테이너 `파일 시스템이 구축`되는 과정
- 파드 속 컨테이너의 파일 시스템은 여러 출처를 합쳐 구성
    - 컨테이너 이미지가 파일 시스템의 초기 내용 제공
    - 이 위에 기록 가능한 레이어(writable layer)가 얹힌다.
    - 도커 이미지는 읽기 전용
    - 이미지에 들어 있던 파일을 수정한다면, 이건 기록 가능 레이어에서 해당 파일의 사본을 수정하는 것
- 애플리케이션은 그저 읽기 쓰기가 가능한 파일 시스템으로만 보임
    - 쿠버의 파일 시스템에 맞춰서 애플리케이션을 수정할 필요 없이, 그냥 그대로 쿠버로 이주시키기 편함
- 파드 속 컨테이너의 생애주기는 해당 컨테이너의 생애주기를 따른다. 그리고 파드의 재시작은 컨테이너의 재시작
    - 데이터가 파드 수준에서 저장되지 않은 상태에서 파드를 재시작하면 데이터는 유실 됨
- 읽기 전용인 컨피그맵, 비밀값 외에도 기록 가능한 볼륨 존재
- __`빈 디렉터리로 초기화`되는 유형의 `볼륨` - `공디렉터리(EmptyDir)`__
```yaml
# deployment 일부
    spec:
      containers:
        - name: sleep
          image: kiamol/ch03-sleep
          volumeMounts:
            - name: data
              mountPath: /data
      volumes:
        - name: data
          emptyDir: {}
```
- 공디텍터리는 파드 수준의 스토리지
    - 이미지나 컨테이너 레이어에 속하지 않음
- 파드가 재시작되더라도 유지된다.
- 새로운 컨테이너도 이전에 같은 파드에 있었던 컨테이너가 기록한 데이터에 접근 가능
- 임시 사용 목적의 로컬 캐시에 적합!!
    - 컨테이너 죽어도 다음 컨테이너가 그대로 읽을 수 있음
- __파드가 대체되어 새 파드를 만들면 처음 상태인 빈 디렉터리가 된다.__
    - 공디렉터리 볼륨은 파드와 생애 주기를 함께하기 때문

## 2. 볼륨과 볼륨 마운트로 `노드에 데이터 저장`하기
- 데이터를 특정 노드에 고정시킬지 말지 결정
- 호스트경로(HostPath) 볼륨
    - 노드의 디스크를 가리키는 볼륨
    - 파드에서 정의
    - 컨테이너 파일 시스템에 마운트되는 형태로 사용
- __`노드의 특정 디렉터리`를 가리키는 `볼륨`__
```yaml
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
            - name: cache-volume # 볼륨 리소스의 이름
              mountPath: /data/nginx/cache # 컨테이너 내의 볼륨이 마운트될 경로
      volumes:
        - name: config
          configMap:
            name: pi-proxy-configmap
        - name: cache-volume
          hostPath: # hostPath 유형의 볼륨 - 노드의 디렉터리를 사용해서 볼륨을 만들겠다!
            path: /volumes/nginx/cache # 사용할 노드의 디렉터리
            type: DirectoryOrCreate # 디렉터리가 없으면 생성하는 type
```
- __파드가 항상 `같은 노드에서 동작하는 한`, `볼륨의 생애 주기`가 `노드의 디스크`와 같아진다.__
    - 새로 생성된 대체 파드는 시작할 때 hostPath 볼륨을 읽어온다.
- __호스트 경로(hostPath)의 문제점__
    - 노드가 두 개 이상인 클러스터일 때!! 
        - 다른 노드에 배치되는 경우 이전에 저장한 데이터를 사용할 수 없음
    - 보안 취약!! 
        - 노드의 파일 시스템 전체에 파드 컨테이너가 접근 가능해짐
        - 노드 디스크 전체를 장악당할 수도!
        - __노드의 파일 시스템 전체에 접근할 수 있는 파드__
        ```yaml
            spec:
                containers:
                    - name: sleep
                      image: kiamol/ch03-sleep
                      volumeMounts:
                        - name: node-root
                        mountPath: /node-root
                volumes:
                    - name: node-root
                      hostPath:
                        path: / # 노드 파일 시스템의 루트 디렉터리를 볼륨으로 지정
                        type: Directory # 경로에 디렉터리가 존재해야 하는 type
        ```
- __그럼 hostPath는 절대 사용하면 안되는건가?__
    - 실행 중 노드의 특정 경로에 접근해야 하는 경우에 필요할 수 있음
    - 이런 경우, 필요 이상으로 노출하지 않는 식으로 안전하게 사용하면 됨
    ```yaml
        spec:
            containers:
                - name: sleep
                  image: kiamol/ch03-sleep
                  volumeMounts:
                    - name: node-root
                      mountPath: /pod-logs
                      subPath: var/log/pods # 볼륨 내 경로
                    - name: node-root
                      mountPath: /container-logs
                      subPath: var/log/containers # 볼륨 내 경로
            volumes:
                - name: node-root
                  hostPath:
                    path: /
                    type: Directory
    ```
    - 볼륨 내 경로를 `subPath`로 지정하여 그 이상의 경로에 접근을 막음
- __호스트 경로의 유용성__
    - 유상태(stateful) 애플리케이션 도입 시 유리
    - 상태가 임시적인 경우!!
    - 상태가 영구적이어야 한다면, `임의의 노드에서 접근 할 수 있는 유형의 볼륨`을 사용해야 한다.

## 3. 전체에서 접근 가능하도록 데이터 저장하기 - `영구볼륨`과 `클레임`
- 분산 스토리지 시스템
    - k8s는 클러스터 전체에서 접근 가능한 스토리지만 제공
- 볼륨 유형 중 분산 스토리지 시스템의 지원을 받는 것
    - AKS 클러스터 - 애저 파일스, 애저 디스크
    - EKS 클러스터 - Elastic Block Store
    - 온프레미스 - NFS(Network File System), GlusterFS 등
- 각각 설정이 달라서, 파드 정의에 이 설정을 기술하게 되면 의존도가 높아짐
- `영구볼륨(PersistentVolume, PV)`와 `영구볼륨클레임(PersistentVolumeClaim, PVC)`
    - __`스토리지 계층`의 `추상`__
        - 참고로 파드는 컴퓨팅 계층의 추상이고 서비스는 네트워크 계층의 추상
    - 스토리지 솔루션과의 `느슨한 결합`을 위해 탄생한 수단
- __영구볼륨 (`PersistentVolume`, `PV`)__
    - 사용 가능한 스토리지 조각을 정의한 쿠버네티스 리소스
    - 각 영구볼륨에는 이를 구현한 스토리지 시스템에 대한 볼륨 정의가 있음
    ```yaml
    apiVersion: v1
    kind: PersistentVolume
    metadata:
        name: pv01 # 볼륨 이름
    spec:
        capacity:
            storage: 50Mi # 볼륨 용량
        accessModes:
            - ReadWriteOnce # 파드 하나에서만 사용 가능
        nfs:
            server: nfs.my.network # NFS 서버의 도메인 네임
            path: "/kubernetes-volumes" # 스토리지 경로
    ```
    - 접근 유형과 용량이 지정되어 있음
    - 하지만 바로 pod가 사용할 수는 없음
        - 영구볼륨클레임을 통해 `볼륨 사용을 요청`해야 함_
- __영구볼륨클레임 (`PersistentVolumeClaim`, `PVC`)__
    - __파드가 사용하는 스토리지의 추상__
        - 포트 어댑터!!!
            - 영구볼륨클레임이 포트, 영구볼륨이 어댑터
            - 파드 입장에서 영구볼륨클레임으로 요청
            - 영구볼륨 입장에서 영구볼륨 클레임으로 사용됨
    - 매니페스트
    ```yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
        name: postgres-pvc
    spec:
        accessModes: # 접근 유형은 필수 설정
            - ReadWriteOnce
        resources:
            requests:
                storage: 40Mi # 요청하는 스토리지 용량
        storageClassName: "" # 스토리지 유형을 지정하지 않음
    ```
    - 이렇게 영구볼륨을 명시적으로 생성해야 하는 방식은 `정적 프로비저닝 방식`
        - 아래에서 영구볼륨을 명시적으로 생성하지 않는 동적 프로비저닝 방식을 다룰 것
    - 접근유형(accessModes), 스토리지 용량(resources.requests.storage), 스토리지 유형(storageClassName) 정의
        - `스토리지 유형 지정을 안하면` 현존하는 영구볼륨 중 요구 사항과 일치하는 것을 찾아준다.
    - 일치하는 영구볼륨을 찾으면, 영구볼륨클레임은 이 영구볼륨과 연결
        - 영구볼륨은 다른 영구볼륨클레임과 추가로 연결될 수 있다.
    - 못 찾으면, 없으면,
        - 대기 상태 - 보류, pending - 가 된다.
        - 이렇게 되면, 이 영구볼륨클레임을 사용하는 Pod도 정상적으로 시작되지 않는다. Pod도 보류상태!!
        - __영구볼륨클레임을 사용하는 Pod 정의__
        ```yaml
        spec:
            containers:
                - name: db
                  image: postgres:11.6-alpine
                  volumeMounts:
                    - name: data
                      mountPath: /var/lib/postgresql/data
            volumes:
                - name: data
                  persistentVolumeClaim:
                    claimName: postgres-pvc # 영구볼륨클레임 이름
        ```
    - 영구볼륨은 클러스터의 인프라와 관계된 것!! - 쿠버네티스의 몫이 아니다.
- __결합도__
    - 영구볼륨클레임을 사용해서 데이터베이스 파드와 웹 애플리케이션 파드의 결합도를 낮추었다!
    - 웹 애플리케이션 파드는 데이터베이스 파드와 무관하게 업데이트 하거나 스케일링이 가능
    - 데이터베이스 파드는 웹 애플리케이션 파드와 무관하게 영구 스토리지를 얻었다.

## 4. `스토리지의 유형`과 `동적 볼륨 프로비저닝`
- 정적 볼륨 프로비저닝
    - 명시적으로 영구볼륨과 영구볼륨클레임을 생성해서 연결
    - 모든 k8s에서 사용 가능
    - 스토리지 접근 제약이 큰 조직에서 선호
- 동적 볼륨 프로비저닝
    - 대부분 k8s에서 지원
    - 간단함
    - `스토리지 유형`만 지정하면 끝
- __동적 영구볼륨클레임 매니페스트(`기본 스토리지 유형`)__
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc-dynamic
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
      # storageClassName 필드가 없으면 기본 유형으로 쓰여진다.
```
- __영구볼륨클레임만 만들면 된다__
    - `영구볼륨클레임`에 영구볼륨을 따로 만들지 않아도 배치
    - 쿠버네티스 플랫폼이 여기 정의된 스토리지 유형에 맞는 영구 볼륨을 생성해서 영구볼륨클레임에 연결해줄 것
- 영구볼륨클레임이 삭제되면 영구볼륨도 함께 삭제
- 스토리지 유형
    - 표준 쿠버네티스 리소스로 생성
    - `스토리지 유형 정의`에 다음 `세 가지 필드`로 스토리지 `동작 지정`
        - `provisioner`
            - 영구볼륨을 만드는 주체. 플랫폼에 따라 잘라짐
        - `reclaimPolicy`
            - 클레임 삭제시 남은 볼륨을 어떻게 처리할지 결정
        - `volumeBindingMode`
            - 영구볼륨클레임이 생성되고 바로 생성해서 연결할지, 영구볼륨클레임을 사용하는 파드가 생성될 때 영구볼륨을 생성할지 선택
- 사용자 정의 스토리지 유형을 사용한 영구볼륨클레임
```yaml
spec:
  accessModes:
    - ReadWriteOnce:
  storageClassName: jm # 스토리지 유형 지정
  resources:
    requests:
      storage: 100Mi
```
- `스토리지 유형`은 `스토리지의 추상`이다.

## 5. 스토리지 선택시 고려할 점
- 데이터베이스 같은 유상태 애플리케이션도 쿠버네티스에서 실행해야 하는가
- 데이터 관리는 쿠버네티스에서도 쉬운 일이 아니다.
- 유상태 애플리케이션, 데이터 백업, 스냅샷, 복원 등 고려해야 하는 것이 많다.
- 클라우드 환경 - `매니지드 데이터베이스 서버`
    - 걍 이거 써~
