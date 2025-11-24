# context 이해하기

- cluster, context, namespace, users... 헷갈림


## config get- 명령어

kubectl config get- 하고 탭을 눌러서 자동완성을 확인하면 아래 것들 나옴

```
$ kubectl config get-
get-clusters  -- Display clusters defined in the kubeconfig
get-contexts  -- Describe one or many contexts
get-users     -- Display users defined in the kubeconfig

```

여기서 ??? 함..
저게 다 뭐지?

k8s config 파일을 보면 차이가 쉽게 이해 됨


## config 파일확인

`~/.kube/config`를 열어 보면 4가지 1레벨 정의가 있음
- clusters, users, contexts, current-context

```
apiVersion: v1
kind: Config
clusters:
- name: my-cluster
  cluster:
    server: https://123.45.67.89:6443
    certificate-authority-data: ...
users:
- name: admin-user
  user:
    client-certificate-data: ...
    client-key-data: ...
contexts:
- name: admin@my-cluster
  context:
    cluster: my-cluster
    user: admin-user
current-context: admin@my-cluster
```

- clusters: 
    - 접근할 cluster들
    - 어디에 접속할 것인가
- users: 
    - cluster에접근하는 user들
    - 누구로 접속할 것인가
- contexts: 
    - 어떤 cluster, user를 사용할지 미리 정의한 세트
- current-context: 
    - 현재 사용중인 context. 여기 정의된 cluster의 api server에 요청


## 네임스페이스는 뭐지?

클러스터 내부에서 또 다시 나눠 놓은 운영 영역 - 리소스 격리
이건 kubectl이 관리(config 파일에서)할 게 *아니라* 클러스터에 정의된 것
namespace도 pod, deployment와 같은 `리소스`임

## context는 말 그대로 문맥

kubectl 이라는 client가 어떤 cluster에 어떤 user로 kube API Server에 요청할 것인지에 대한 정보 묶음

k8s cluster의 API Server와 kubectl이라는 client 이 둘을 구분하는 것 부터 시작!

그럼 context는 자연스럽게 API Server와 통신을 위한 client의 정보 묶음이라고 이해됨

여러 클러스터, 여러 유저가 존재할 수 있기에, 이 중에 뭐를 선택해서 API Server에 요청을 할 것인지 묶어서 관리
매번 어떤 클러스터, 어떤 유저를 사용할거라고 하기 귀찮으니까, context로 한번에 세팅해서 사용

context도 여러 개 정의 가능하고, current-context로 context를 집어서 사용
