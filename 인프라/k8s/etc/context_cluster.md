# context와 cluster 차이
- kubectl config get-context와 kubectl config clusters
- 이 두 명령어가 같은 리스트를 출력하는 것 같아서, context와 cluster의 차이가 궁금
- 간단히
    - context는 작업 환경 정의 
    - cluster는 클러스터 자체 정의

## cluster
- 쿠버네티스의 리소스(Pods, Nodes, Services 등)를 관리하는 하나의 쿠버네티스 인스턴스
- 구성 요소
    - API 서버 주소
        - 쿠버네티스의 클러스터와 통신하기 위한 엔드포인트
    - CA 인증서
        - 클러스터와의 보안 통신을 위한 인증서
- 역할
    - 클러스터는 쿠버네티스 리소스를 관리하는 물리적/논리적 환경
    - 한 클러스터에서 노드와 파드를 직접 관리
- kubeconfig 예시 (kubectl config view 로 확인)
```yaml
clusters:
- name: dev-cluster
  cluster:
    server: https://dev-cluster.example.com
    certificate-authority-data: LS0tLS1...
- name: prod-cluster
  cluster:
    server: https://prod-cluster.example.com
    certificate-authority-data: LS0tLS1...
```

## context
- __`작업 대상 쿠버네티스 클러스터`와 `kubectl 명령에서 사용할 기본 네임스페이스`를 정의__
    - 현재 작업 대상으로 보는 클러스터
    - 현재 작업 대상으로 보는 네임스페이스
    - 컴퓨터 사이언스에서 `context`라는 단어가 주는 의미 생각!!
- 쿠버테니스 클러스터와 통신할 때 사용할
    - `클러스터`, `사용자`, `네임스페이스` 정보를 묶어 놓은 것
- 사용자는 여러 클러스터를 관리할 수 있음
- 또한 클러스터에서 다른 네임스페이스를 다룰 수도 있음
- context 구성요소 확인
    - __`kubectl config get-contexts`__
- 구성요소
    - cluster
        - 어떤 클러스터와 연결할지
        - context와 cluster는 다대일 관계
        - 하나의 cluster를 여러 context가 참조 가능
        - 어떤 클러스터 환경에 대한 작업 환경 설정을 context가 한다고 보면 될듯
        - context의 cluster를 바꿀 수 있음
            - __`kubectl config use-context {새 클러스터 이름}`__
        - context의 현재 cluster 확인
            - __`kubectl config current-context`__
    - user (Authinfo)
        - 클러스터 연결시 사용할 사용자 계정 정보
        - 현재 context의 유저 변경
            - __`kubectl config set-context --current --user=new-user`__
        - 특정 context의 유저 변경
            - __`kubectl config set-context my-context --user=new-user`__
    - namespace
        - 명령어로 지정하지 않으면 default 네임스페이스 사용
        - context의 default namespace를 바꿀 수 있음
            - __`kubectl config set-context --current --namespace={네임스페이스_이름}`__

## 왜 컨텍스트가 있어야 하는가?
- `유연한 작업 환경 제공`
- 동일한 클러스터에 대해 여러 작업 환경을 분리해서 사용 가능
- dev, prod, test 등의 환경을 분리해서 작업 환경 구축 가능
- kubeconfig 예시 (kubectl config view 로 확인)
```yaml
contexts:
- name: dev-context
  context:
    cluster: dev-cluster
    user: dev-user
    namespace: default
- name: prod-context
  context:
    cluster: prod-cluster
    user: prod-user
    namespace: production
- name: test-context
  context:
    cluster: dev-cluster
    user: test-user
    namespace: test
```