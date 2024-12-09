# minikube

## 클러스터 구축
```shell
% minikube start --nodes 2 -p jm
```
- 옵션
    - --nodes 2
        - 노드 개수 지정
    - -p jm
        - minikube 클러스터의 프로필 이름 설정
- 이후에는 생성한 cluster가 default namespace로 설정
    - kubectl이 적용되는 default cluster!!

## 상태 확인
```shell
% minikube status -p jm
```

## 클러스터 리스트 확인
```shell
$ minikube profile list
```

## 중지
```shell
$ minikube stop -p jm
```

# 삭제
```

```