# Service
- 파드 통신
    - 표준 네트워크 프로토콜인 TCP와 UDP 지원
        - IP 주소로 트래픽을 제어
- 문제는 IP 주소가 파드가 바뀔 때 주소가 바뀌어 버린 다는 것
- Service라는 리소스는 address discovery 기능을 제공
    - `address discovery`??
        -  네트워크나 분산 시스템에서 사용자가 찾고자 하는 장치, 서비스, 노드 또는 리소스의 네트워크 주소(IP 주소, 포트 등)를 동적으로 식별하거나 확인하는 과정
    - 요약 하면 `동적으로` 주소를 찾아준다는 것
- pod에서의 통신 트래픽의 라우팅 처리

## 쿠버네티스 내부 트래픽 라우팅
- pod는 쓰고 버리는 리소스
    - 뭔가 문제가 생겼을 때나 새롭게 배포할 때 pod를 수정하는 게 아니라 새 pod로 교체 해버림
- 문제는 pod가 새롭게 만들어지면 ip 주소가 다시 설정 됨
- 이 ip는 쿠버네티스 API를 통해 파악 가능
```bash
kubectl get pod -l {label_key}={label_value} \
    --ouput jsonpath='{.items[0].status.podIP}'
```
- k8s의 가상 네트워크(CNI, Container Network Interface)는 클러스터 전체를 커버
    - pod는 ip 주소를 통해 서로 통신할 수 있음
    - pod가 생성될 때 ip 부여
    - **보통 이 ip로 직접 통신하지 않는다.** pod가 새롭게 생성될 때마다 ip가 바뀌니까
- 언제든지 다른 것으로 바뀔 수 있는 리소스에 접근하기 위한 고정된 주소
    - 도메인 네임!!
    - 쿠버네티스 클러스터에도 전용 DNS 서버가 있음
    - `"서비스의 이름: IP주소"` 이런 구조로 구성
- **서비스란?**
    - **`파드`가 가진 `네트워크 주소`를 `추상화`**
    - 서비스는 자신만의 IP 주소를 가짐
    - 서비스가 삭제될 때까지 바뀌지 않음
    - 서비스와 연결된 파드의 실제 IP 주소로 요청을 연결해줌
    - 레이블 셀렉터로 Pod를 찾음
- 이렇게 서비스가 네트워크 주소를 추상화해서 관리하면 pod는 더 유연하게 교체될 수 있음
    - pod가 교체되어도 서비스가 그대로라면, 이전의 도메인으로 똑같이 pod에 접근 가능
    - 이게 가능한 이유는 pod와 service가 label을 통해 `느슨한 결합`을 하기 때문
- **기본 메니페스트**
```yaml
apiVersion: v1 # core api group
kind: Service
metadata:
    name: jm # 서비스의 이름이 도메인의 이름으로 사용됨
spec:
    selector:
        app: jm # 연결할 pod의 label
    ports:
        - port: 80 # 서비스의 port, targetPort를 생략하면 port를 targetPort로 사용
```
- 서비스의 이름이 도메인 네임으로 사용됨
- 쿠버네티스 가상 네트워크 안에 있는 pod 끼리는 도메인 네임으로 통신 가능
- **서비스 디스커버리**
    - 서비스 리소스 배포
    - 서비스 이름을 도메인 네임으로 사용
    - 다른 컴포넌트와 통신

## 파드와 파드 통신
- 서비스는 여러 개의 type을 가진다.
    - 그 중 가장 기본은 **`ClusterIP`**
        - default type임
        - 그래도 명시하는 게 좋음
    - 클러스터 전체에서 통용되는 IP 주소
    - 같은 클러스터라면, 어느 노드에 있더라도 접근 가능
    - 외부에서는 ClusterIP에 접근할 수 없음 **(중요!!!)**
- **메니페스트**
```yaml
apiVersion: v1 # core api group
kind: Service
metadata:
    name: jm # 서비스의 이름이 도메인의 이름으로 사용됨
spec:
    selector:
        app: jm # 연결할 pod의 label
    ports:
        - port: 80 # 서비스의 port, targetPort를 생략하면 port를 targetPort로 사용
    type: ClusterIP # default type
```
- 서비스를 베포하면 쿠버네티스 DNS 서버에 도메인 등록
    - pod, deployment 변경 없이, 그저 service만 추가로 배포하면 끝
- 서비스를 그대로 두면, 
    - deployment나, pod를 새롭게 배포하더도
        - 그래서 ip가 새롭게 부여되더라도
    - label만 일치하게 만들면 통신에 문제가 없다.
- **서비스는 pod의 네트워크를 추상화한 것**
    - pod 교체는 자주 일어남
        - 뭔가 변경이 일어나면 pod를 교체해버림
    - 그래서 ip가 변함
    - 그럼에도 통신에 문제가 없음
    - label을 통한 느슨한 결합

## 외부 트래픽 파드로 전달
- **`LoadBalancer`(로드밸런서)**
    - 서비스의 유형(type) 중 하나
    - 로드 밸런서가 위치한 노드가 아니더라도, 어떤 노드에라도 트래픽을 전달할 수 있음
    - 어떤 노드에 배치할지 고민할 필요 없음. label selector와 일치하는 파드가 많아도 알아서 연결해줌
    - 외부 로드밸런서
        - 클러스터로 트래픽을 전달해주는 외부 로드밸런서와 함께 동작
        - `포트-어댑터 패턴`
            - 외부 로드밸런서가 구현체, LoadBalancer 타입의 서비스가 포트
    - 레이블 셀렉터와 일치하는 pod로 트래픽 전달해줌
    - 메니페스트
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
        name: jm-web
    spec:
        ports:
            - port: 8080 # 서비스의 포트
                targetPort: 80 # pod의 포트
        selector:
            app: jm-web # 이 레이블을 가진 pod에 트래픽 전달
        type: LoadBalancer # type
    ```
    - **서비스 상태 조회**
        - kubectl get svc jm-web
            - 서비스 상세 정보
        - kubectl get svc jm-web -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8080'
            - 앱의 URL을 EXTERNAL-IP 필드로 출력
    - **`EXTERNAL-IP`**
        - 외부 IP 주소
        - 클러스터에서 제공되는 주소
        - 외부에서는 이 IP 주소로 접근 한다.
        - 쿠버네티스 플랫폼 종류에 따라 외부 IP 주소는 달라진다.
            - 도커 데스크톱
                - loadhost
            - k3s
                - 호스트 컴퓨터의 IP. locahost 또는 로컬 IP 주소
            - AKS, EKS
                - 공인 IP 주소
- **`NodePort`**
    - 유연하지 않은 서비스 유형(type)
    - 모든 노드가 NodePort 타입의 서비스를 바라보다가 해당 서비스로 트래픽이 들어오면, service의 label selector와 일치하는 pod로 트래픽 전달
    - 로드 밸런싱 X
    - 바로 해당 pod로 트래픽 전달
    - 메니페스트
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
        name: jm-web
    spec:
        ports:
            - port: 8080 # 다른 pod가 서비스에 접근하기 위해 사용되는 포트
              targetPort: 80 # pod의 포트
              nodePort: 30001 # 서비스가 외부에 공개되는 포트
        selector:
            app: jm-web # 이 레이블을 가진 pod에 트래픽 전달
        type: NodePort # type
    ```
- port 필드의 차이 
    - 로드밸런서 유형에서는 외부와 내부 모두에서 접근할 때 사용되는 포트
    - 노드포트 유형에서는 내부에서 접근할 때 사용되고, 외부에서 접근할 때에는 nodePort에 적힌 포트로 접근해야 함

## 외부로 트래픽 전달
- 데이터베이스 같은 스토리지 컴포넌트는 쿠버네티스 외부에서 실행하는 경우 많음
- 클러스터 외부를 가리키는 도메인 네임 resolving에도 서비스 리소스를 사용

- **`ExternalName (익스터널 네임)`**
    - 서비스 유형
    - 어떤 도메인 네임에 대한 별명
    - 파드에서는 로컬 네임을 사용하고, 이 로컬 네임으로 접근했을 때 쿠버의 DNS 서버가 외부 도메인으로 해소
    - **`느슨한 결합`**
        - 외부 IP에 직접 접근하는 게 아니라, 내부에서만 사용하는 별칭으로 접근
        - 모든 시스템이 별칭을 통해 구현되어 있어서, 외부 환경이 바뀌더라도 유연하게 변경 가능 (Open-Closed Good)
        - ex) 환경에 따라 직접 데이터베이스에 접근할 수도 있고, 테스트 환경에 접근할 수도 있음
- 메니페스트
```yaml
apiVersion: v1
kind: Service
metadata:
  name: jm-api
spec:
  # selector 없음. 그저 pod들이 jm-api로 접근시 raw.githubusercontent.com로 트래픽을 전달해줄 뿐!
  type: ExternalName
  externalName: raw.githubusercontent.com
```
- 내부 도메인 네임인 `jm-api`로 접근하면 쿠버의 dns는 `raw.githubusercontent.com`라는 `CNAME`을 반환
    - CNAME이란? 
        - 한 도메인 네임을 다른 이름으로 매핑시키는 도메인 시스템의 리소스 레코드 중 하나
        - 도메인 네임 시스템은 진짜 리소스의 주소를 다른 이름으로 매핑해주고, 이 매핑된 이름으로 접근했을 때 진짜 주소를 반환해줌
        - 도메인 네임으로 접근했을 때 반환해주는 리소스의 이름의 종류가 여러개인데, 그 중 cname이 있음
        - 도메인 주소를 또 다른 도메인 주소로 매핑해서, 도메인 네임으로 접근했을 때 반환 되는 진짜 이름이 ip가 아니라 또 다른 도메인 네임인 경우
        - 이런 경우를 CNAME이라고 함
- 애플리케이션이 사용하는 주소가 가리키는 대상을 치환해줄 뿐, 요청 내용을 못 바꿈
    - TCP는 상관 없지만, HTTP 서비스는 문제
    - 헤더에 대상 호스트명이 들어가는데, 헤더의 호스트명이 응답의 호스트네임과 다르면 HTTP 요청이 실패
    - HTTP 요청이 아니라면 ExternalName 굳굳

- **`헤드리스 서비스(headless service)`**
    - 마찬가지로 `도메인 네임` 대신 `IP 주소`를 대체해 주는 방법
    - **클러스터 내부의 가상 IP로 대체**
        - *서비스의 이름인 도메인 네임으로 내부 pod가 요청을 날리면, endpoint에 적힌 ip로 트래픽이 전달됨*
    - 클러스터 IP 형태로 제공
    - 레이블 셀렉터가 없음
        - 대상 파드 없음
    - IP 주소의 목록이 담긴 엔드포인트 리소스와 함께 배포
- 메니페스트
```yaml
apiVersion: v1
kind: Service
metadata:
  name: jm-api
spec:
  # selector가 없음
  # 대신에 Endpoints 리소스와 엮임
  type: ClusterIP
  ports:
    - port: 80
---
kind: Endpoints
apiVersion: v1
metadata:
  name: jm-api
subsets:
  - addresses: # 정적 IP 주소 목록
      - ip: 192.168.123.234 # jm-api로 접근하면 이 ip로 트래픽 전달
    ports:
      - port: 80 # 각 IP 주소에서 주시할 포트
```

## 서비스의 해소 과정
- 내부 동작
    - Pod에서 나오는 `모든 통신`은 쿠버네티스의 구성 요소 중 하나인 `네트워크 프록시`가 라우팅을 담당
    - 이 프록시는,
        - 각 노드에서 동작
        - 모든 서비스의 엔드포인트에 대한 최신 정보 유지
            - __서비스의 엔드포인트를 설정하는 방법__
                - `label selector`
                    - pod가 endpoint
                - `헤드리스 서비스`
                    - Endpoint 리소스에서 정의한 ip가 엔드포인트
                        - 외부일수도 있고 내부일수도 있음
                - `externalName`
                    - 외부 도메인 네임이 엔드포인트
        - 네트워크 패킷 필터(구현체는 플랫폼마다 다름, 리눅스는 IPVS 또는 iptables)를 사용하여 트래픽을 라우팅
- __`ClusterIP`__ 는 네트워크상에 실재하지 않는 __`가상 IP 주소`__ 임
    - 모든 통신은 프록시를 통함
    - __ClusterIP로 접근하면 프록시를 통해서 실제 엔드포인트로 연결 됨__
        - 여기서 엔드포인트는 외부일 수도 있고, 내부 일수도 있음
    - 서비스 컨트롤러
        - 파드가 변경되어 엔드포인트가 바뀌면, 컨트롤러가 네트워크 프록시의 엔드포인트 목록을 바꿔준다.
        - 그래서 가상 정적 IP와 네트워크 프록시만 있으면 항상 최신의 엔드포인트 목록을 적용 받는다.
    - 정적 가상 IP 주소는 파드가 교체되더라도 그대로 유지
        - DNS 조회 결과를 영구적으로 캐시 가능
- __의문 1: 왜 DNS 조회 결과가 엔드포인트의 IP가 아니라 클러스터의 IP 주소인가__
    - 쿠버의 DNS 서버는 엔드포인트 IP 주소가 아닌 클러스터의 IP 주소를 반환함.
    - 엔드포인트는 변하는 요소이기 때문
- __의문 2: 도메인 네임은 왜 .default.svc.cluster.local 과 같이 끝나나.__
    - `로컬 도메인 네임`은 `네임스페이스를 포함`하는 완전한 도메인 네임의 별명!
- __`Namespace`__
    - 모든 쿠버네티스 리소스는 네임스페이스 안에 존재
    - 네임스페이스도 리소스
    - 클러스터를 논리적 파티션으로 나눔
    - DNS 해소 과정에 네임스페이스 관련
        - 서비스의 도메인 네임은 서비스가 속한 네임스페이스 안에서 유효
        - __다른 네임스페이스에 있는 서비스에 접근할 땐, 네임스페이스도 같이 표기해주어야 함__
            - ex) jm-api.default.svc.cluster.local
    - `--namespace`
        - kubectl get svc --namespace default
        - kubectl get svc -n default
        - 이렇게 네임스페이스를 사용해서 다른 네임스페이스를 대상으로 지정 가능

## 정리
- Pod는 IP 주소를 가짐
- TCP/UDP 프로토콜로 통신
- DNS 디스커버리 기능으로 파드의 IP 주소를 직접 참조 안함. 서비스를 통해 접근
- 파드 간 통신, 외부에서 파드로, 파드에서 외부로
- 서비스는 파드나 디플로이먼트 생애주기와는 별개
