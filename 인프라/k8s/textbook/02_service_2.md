# service 고찰

## port와 targetPort
- port
    - Service에서 사용하는 port
    - 클라이언트가 이 포트를 통해 service에 요청을 보냄
    - 클라이언트 -> service 간의 연결 포트
- targetPort
    - pod에서 실제로 요청을 처리할 컨테이너의 포트
    - service가 트래픽을 라우팅할 pod 컨테이너에서 열려 있는 포트
    - service -> pod 간의 연결 포트

## port만 쓰는 경우가 있고, targetPort를 쓰는 경우가 있음
- port만 정의하면, targetPort는 기본적으로 port와 동일한 값을 사용
- Service의 포트와 Container의 포트가 같은 경우 targetPort를 생략할 수 있음

## yaml에서 '-' 는?
- 리스트 형식의 데이터를 표현할 때 각 항목 앞에 -를 붙임
```yaml
ports:
  - port: 80
    targetPort: 8080
  - port: 443
    targetPort: 8443
```

## Port 세가지 유형
- `NodePort` --> `Port` --> `targetPort`
- NodePort 
  - 외부에서 접속하기 위해 사용하는 포트
- Port
  - Service 객체의 포트
    - NodePort 유형의 서비스에서는 내부에서 접근할 때만 사용될 것
    - LoadBalancer 유형의 서비스에서는 외부에서 접근할 때도 사용될 것
- targetPort
  - Service 객체로 전달된 요청을 Pod로 전달할 때 사용하는 포트

- __LoadBalancer와 NodePort에서 port 필드의 차이__
    - 로드밸런서 유형에서는 외부와 내부 모두에서 접근할 때 사용되는 포트
    - 노드포트 유형에서는 내부에서 접근할 때 사용되고, 외부에서 접근할 때에는 nodePort에 적힌 포트로 접근해야 함

## Service의 Endpoint
- service는 selector를 보고 pod의 ip 주소를 Endpoint로 등록 한다.
  - selector가 있으면 pod의 ip로 Endpoint 객체가 자동 생성되는 것
- Endpoint를 수동으로 생성해서 사용하고 싶다면 selector를 안쓰면 됨
- 만약 selector도 쓰고 Endpoint도 수동으로 작성했다면, selector가 우선!!

## 참고
- 쿠버네티스 교과서