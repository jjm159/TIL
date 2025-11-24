# matchLabels는 왜 적지? 

## 의문
- deployment의 메니페스트에서는 selector에 label를 지정할 때, 
- matchLabels 라는 것을 사용
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
    name: jm
spec:
    selector:
        matchLabels: # 이거!!!!
            app: jm-app
# ...
```

그런데 service의 메니페스트에서는 그냥 바로 레이블을 넣어버림
```yaml
apiVersion: v1
kind: Service
metadata:
    name: jm-app
spec:
    selector: # matchLabels를 사용하지 않고 바로 레이블 작성
        app: jm_app
    ports:
        - port: 80
```

## apiVersion의 차이 (스키마 차이)
- 무슨 차이일까 살펴보는데, apiVersion이 다름을 발견!!
- apiVersion은 k8s의 apiserver의 버전임
    - apps/v1
        - apps API 그룹의 v1 버전 의미
        - 사용되는 리소스
            - Deployment, StatefulSet, DaemonSet, replicaSet, ControllerRevision
        - `apps API 그룹은 워크로드 관리(애플리케이션 배포 및 관리)를 위한 리소스들을 관리`
    - v1
        - 쿠버네티스의 Core API Group의 V1 버전 의미
        - 사용되는 리소스
            - Pod, Service, ConfigMap, Secret, PersistentVolume, Namespace, Node
        - API 그룹 이름을 명시하지 않음. 기본, 코어니까.
        - `Pod, Service, ConfigMap 같은 기본 리소스 정의시 사용`
- 이 버전에 따라 selector도 다르게 사용되는 것

## 꼭 mtchLabels(또는 matchExpressions)를 적어야 하고, 꼭 적지 말아야 한다.
- `App API Group(apps/v1)` vs `Core API Group(v1)`
    - `App API Group(apps/v1)`에서의 selector
        - 꼭 `matchLabels` 또는 `matchExpressions`와 같은 구조를 통해 label selector를 정의해야 함
        - 단순 라벨 매칭인 경우 `matchLabels`를 사용
    - `Core API Group(v1)`에서의 selector
        - `matchLabels`이나 `matchExpressions`을 __지원하지 않음__
        - 그래서 바로 label을 적음
- `App API Group(apps/v1)`에서는 라벨 셀렉터 구조를 더 정교하게 다룰 수 있게 설계
    - matchLabels이나 matchExpressions을 통해 라벨 매칭 방식을 확장
    - 그래서 반드시 적어야 하고, 
    - Core API Group인 경우에는 적을 수 없다. 없는 기능이니까

## matchExpressions
- 복잡한 라벨 매칭 조건 정의 가능
- 특정 키가 여러 값 중 하나를 포함하는지, 키가 존재하는지 등을 따져서 select 할 수 있음
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 3
  selector:
    matchExpressions:  # matchExpressions만 사용
    - key: app
      operator: In # 다음 밸류 중 하나면 match~!
      values:
      - my-app
      - another-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: nginx
```
- matchLabels는 정확히 하나의 label과 일치해야 함
