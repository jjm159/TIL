# utm

- Universal Turning Machine
- mac에서 qemu 기반 가상 머신을 쉽게 실행할 수 있게 도와주는 오픈 소스 앱
- 다양한 운영 체제를 가상 머신으로 실행 가능

## 설치
```
brew install --cask utm
```

## 운영체제 설치
- ISO 파일 설치

## 가상 머신 생성
- create a new virtual Machine

## os 유형 설정
- 주의!!! 여기서 삽질함 !
- kernel이나 다른 걸 체크하면 안되고,
- 반드시 Boot ISO Image를 선택하고
- 아까 설치한 os iso 파일의 경로를 설정해 주어야 함

## 네트워크 설정
- 마지막에 확인할 때 edit 하는 거 체크
- 또는 완료하고 오른쪽 클릭해서 edit 화면 진입
- 네트워크 탭에서
    - advanced 설정
        - 여기에 로컬 고정 private 주소를 설정
            - 이 주소로 접근할 것!

## 마무리
- start 
