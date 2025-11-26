# iterm2

## 1. 키매핑

### 문제

기본으로 cmd + {좌, 우}, opt + {좌, 우} 이런 키들이 iterm3에서 입력이 안됨
키가 충돌되어서 또는 매핑이 안되어 있어서 그럼

### 해결 1 - 삭제 

Settings -> Keys -> Key Bindings
여기서 cmd + {좌, 우} 삭제

### 해결 2 - 매핑

Settings -> Profiles -> Keys -> Key Bindings
아래 키들 추가

- opt + delete -> Send Hex Codes: 0x17
- cmd + delete -> Send Hex Codes: 0x15
- cmd + left -> Send Escape Sequence: b
- cmd + right -> Send Escape Sequence: f
- opt + left -> Send Hex Codes: 0x01
- opt + right -> Send Hex Codes: 0x05

속이 후련
