# multipass

- mac os 전용 경량 vm

## 설치
```
brew install --cask multipass
```

## 우분투 vm 생성
```
multipass launch --name k3s-ubuntu --cpus 2 --mem 2G --disk 10G
```

## vm 접속
```
multipass shell k3s-ubuntu
```
