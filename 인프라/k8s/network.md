# k8s 네트워크

## 네트워크 모델 이해하기

- pod 마다 ip를 가져서 클러스터 내부 pod에 바로 접근 가능
- NAT가 없이 모든 pod와 통신 가능하도록 flat하게 만들어서 단순화 함
    - 원래 node는 물리 서버니까 다른 node에 있는 pod에 접근하려면 NAT가 있는게 실제 세상에서 원칙
        - NAT(Network Address Translation):  
    - k8s는 같은 클러스터 안에서는 바로 접근 가능하게 함
- 같은 pod 안에 container들은 동일 ip/port 공유, localhost로 바로 접근 가능

## CNI

- 사용자는 플랫폼에 상관없이 k8s의 네트워크 모델만 이해하고 사용하면 됨
- k8s는네트워크 기능을 CNI라는 통일된 인터페이스로 사용
- 내부 구현을 CNI에 맞게 각 플랫폼(linux, macos, windows)에서 구현

## Service

#### 왜 서비스 필요?
- pod는 가변적이고 일시적 - 언제든 없어질 수 있음
- 다시 생성될 때 마다 ip 새로 부여
- 클러스터 외부에서 해당 pod에 접근시 고정된 IP와 DNS 이름으로 접근 필요
    - Service가 그 역할 
- Service는 pod 그룹으로 로드밸런싱해서 트래픽 전달

#### 왜 이름이 서비스지?

- 여러 pod가 하나의 애플리케이션 역할 수행
- 외부에서 볼 때 하나의 Service로 보임

#### Endpoints
- https://yoo11052.tistory.com/193
- pod ip 집합을 Endpoints라고 함


## Service Type 타입!!

- `ClusterIp 타입 서비스`
    - 기본 서비스 타입. Type 명시 안하면 이거
    - 서비스 생성하면 자동으로 클러스터 내부 가상 ip 부여
    - 이 ip로 접근하면 kube-proxy가 iptables/ipvs를 사용해서 적절한 pod로 라우팅
    - 이걸로 하면 외부 접근 안됨
    - 저 ip는 클러스터 내부 ip!!!

- `NodePort 타입 서비스`
    - 외부에서 접근할 port를 부여해서 외부 접근을 가능하게 하는 서비스 타입
    - 이 타입으로 지정하면 외부에서 이 서비스로 접근 가능
    - Node(물리 컴퓨터) 입장에서 트래픽이 들어오면(node의 ip로), 어떤 서비스로 보낼지 결정해야 함
    - 이 때 서비스를 결정하는 게 포트고, NordPort 타입으로 이 port를 지정함
        - 명시하지 않으면 random하게 자동으로 할당
    - 트래픽 전달 과정
        - https://노드IP:노드Port 접근 -> Node 타입 서비스 -> Pod

- `LoadBalancer 타입 서비스`
    - 퍼블릭 IP를 부여 받는 서비스 타입
    - LoadBalancer 타입 서비스로 선언하면,
        - 클라우드 Provider의  L4 로드밸런서 자동 생성
        - 그리고 이 로드밸런서에는 퍼블릭 IP 부여
    - 로드밸런서의 퍼블릭 IP로 접근하면, 
        - 그 후부터는 NodePort가 동작
        - 그냥 NodePort 타입까지는 퍼블릭 IP 부여가 보장되지 않는데,
          LoadBalancer 타입은 퍼블릭 IP가 부여됨. 걍 이게 다인듯

- `ExternalName 타입 서비스`
    - CoreDns에 해당 서비스 네임에 외부 도메인 네임을 매핑해주는 서비스 타입
    - pod가 서비스 네임으로 요청하면 DNS 질의를 하는데, 
    - 이 서비스 타입이 ExternalName이면 외부 도메인 네임을 반환함
    - 그럼 pod는 이 외부 도메인으로 요청을 보내게 됨

## CoreDNS
- k8s 내부에 dns 서버가 있음
- 서비스가 생성되면 자동으로 dns 서버에 이름과 ip 등록
- 클러스터 내부에서 service name으로 요청하면 이 dns 서버로부터 ip를 받아와서 요청
- 다른 네임스페이스인 경우에는 fullname으로 요청해야 함
- 이름 규칙
    - 서비스이름.네임스페이스.svc.cluster.local
- 왜?
    - pod나 service의 ip가 바뀌어도 이름으로 일관된 통신 가능

## HostPort

- HostPort는 특정 pod에 접근 하기 위한 port를 명시해서 바로 해당 pod로 접근 가능하게 함
- NodePort는 특정 service로 라우팅, HostPort는pod로 라우팅
- NodePort는 service로 라우팅해서 어떤 pod로 전달될지 모름
