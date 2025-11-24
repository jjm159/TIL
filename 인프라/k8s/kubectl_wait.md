# kubectl wait
- kubectl wait 명령어는 쿠버네티스 리소스가 특정 `조건(condition)`을 만족할 때까지 대기하는 데 사용
- kubectl wait --for=condition=Ready pod <pod-name>
    - 특정 pod가 Ready 상태가 될 때까지 대기한다는 의미

## 왜 kubectl wait를 사용하는가?

#### 1. 리소스의 상태를 확인한 후 작업을 실행하기 위해
- 쿠버네티스 리소스(Pod, Deployment 등)는 생성되었더라도,준비가 완료되지 않은 상태일 수 있다.
- Ready 상태가 되기 전에 다른 작업을 실행하면 오류가 발생하거나 의도한 대로 동작하지 않을 수 있다.
- kubectl wait는 리소스가 특정 조건을 만족한 후 작업을 실행할 수 있도록 보장

#### 2. 자동화 및 스크립트에서 사용 -> 거의 여기서 사용할 듯!!
- CI/CD 파이프라인이나 배포 스크립트에서 리소스 준비 상태를 기다리기 위해 사용
- __Pod가 Ready 상태가 되기 전에 다른 단계로 진행하는 것을 방지__

#### 3. 비동기 작업 동기화
- 쿠버네티스는 비동기적으로 리소스를 생성하고 상태를 업데이트
- kubectl wait를 사용해 필요한 상태가 될 때까지 대기

## Ready Condition의 의미
- Pod의 Ready 상태는 쿠버네티스가 해당 Pod가 정상적으로 애플리케이션을 실행할 준비가 되었다는 것을 의미
- Pod의 Readiness Probe 설정에 따라 결정되며, kubectl describe pod에서 확인 가능한
- 확인하기
```bash
kubectl get pod <pod-name>
```

## 사용하기
- kubectl wait --for=condition=Ready pod <pod-name>
    - Ready 상태가 될 때까지 기다림
- kubectl wait --for=condition=Ready pod <pod-name> --timeout=30s
    - 기다리는데, 최대 30초까지
- kubectl wait --for=condition=Ready pods --all
    - 여러 pod가 ready 상태가 될 때까지 대기