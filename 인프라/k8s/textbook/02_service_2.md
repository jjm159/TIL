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

## 참고
- 쿠버네티스 교과서