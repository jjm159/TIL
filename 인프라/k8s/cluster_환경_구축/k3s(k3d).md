# k3s

- 경량화된 쿠버네티스 버전
- 리소스가 제한된 환경에 적합
- 학습 목적으로도 사용하기 좋음

## 설치 전에
- 도커 데스크톱이 설치되어 있어야 함
- kubernetes 탭에서 쿠버네티스 활성화 안해도 된다.

## 설치
- 맥에서는 k3d를 설치해야 함
```bash
brew install k3d
```

## 클러스터 생성 - k3d cluster create
```shell
k3d cluster create jm-cluster
```
- 이후 자동으로 current-context가 k3d으로 클러스터로 바뀜

## kubectl 클러스터 리스트 확인
- 확인
```shell
kubectl config get-clusters
```
- 앞에 `"k3d-"`라는 prefix가 붙어서 cluster가 생성됨!
    ```
    NAME
    k3d-mycluste // 이거!!
    docker-desktop
    elk-tutorial
    ```

## 전체 context 확인


## 현재 kubectl의 current-context 확인
```bash
kubectl config current-context
```
- 현재 default cluster 출력

## current-context로 설정
- cluster를 생성하면, 자동으로 kubectl의 `current-context`가 되지만, 
- 이후에 바뀔 수 있음. 
- 다시 저 클러스터를 `current-context`로 만들고 싶으면 다음 명령어 실행
```bash
kubectl config use-context k3d-my-cluster
```

## 포트 포워딩 설정 - -p 옵션
- 외부에서 접근하려면, 포트를 매핑해야 함
``` bash
k3d cluster create jm-cluster -p "8080:80@load-balancer"
```

## 생성된 클러스터 확인 - k3d cluster list
```bash
k3d cluster list
```

## 삭제 - cluster delete
```bash
k3d cluster delete jm-cluster
```