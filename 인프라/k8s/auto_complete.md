# 터미널 자동완성 설정하기

## 1. macos (zsh)

### kubectl 자동완성 스크립트 실행 (현재 셸에 zsh의 자동 완성 설정)

```
$ source <(kubectl completion zsh)  
```

### 아래 명령어 사용해서 .zshrc에 설정 추가
```
$ echo '[[ $commands[kubectl] ]] && source <(kubectl completion zsh)' >> ~/.zshrc # 자동 완성을 zsh 셸에 영구적으로 추가한다.
```

아래 내용이 `.zshrc`에 추가되어 있으면 됨
```
$ [[ $commands[kubectl] ]] && source <(kubectl completion zsh) # 이게 추가되어 있으면 됨
```

## 2. window (powershell)

### kubectl 자동완성 스크립트 실행 (현재 셸에 zsh의 자동 완성 설정)

```
$ kubectl completion powershell | Out-String | Invoke-Expression
```

### 설정 추가

PowerShell 프로필 파일 수정
아래 내용 추가

```
'kubectl completion powershell | Out-String | Invoke-Expression' >> $PROFILE
```

