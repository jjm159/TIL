# kubectl
- kubectl이 관리하는 클러스터 확인을 위한 명령어들

---

## 클러스터 리스트
- 현재 생성된 클러스터 리스트 확인
```bash
$ kubectl config get-clusters
```

---

## 컨텍스트 
- kubectl이 클러스터와 통신할 때 사용하는 클러스터, 사용자, 네임스페이스 설정

- 현재 컨텍스트
```bash
$ kubectl config current-context
```

- 모든 컨텍스트 확인
```bash
$ kubectl config get-contexts
```

- 특정 클러스터로 전환
```bash
$ kubectl config use-context <context-name>
```

- current-context 확인
```bash
$ kubectl config current-context
```
