# etc
- 환경 설정
- 실행 방법

## 환경 설정
```shell
sudo apt install -y gcc make qemu-system-x86 python3
```
- 이거 하고 시작 해야 한다.

## 테스트 방법
```shell
source ./activate
cd threads
make clea
make
cd build
make test
```
- 이렇게 하면 된다. 

## 하나만 실행

```shell
pintos -- run priority-donate-one
```
- 이렇게 테스트 이름을 run 뒤에 적어준다.

```shell
pintos -T 10 -- -q run priority-donate-one
```
- -T 10 은 10초 이내 안끝나면 종료
- -q는 테스트 종료 후 os 종료

```shell
alias p_dona_one="cd /root/pintos-kaist;source ./activate;cd threads;make clean;make;cd build;pintos -T 10 -- -q run priority-donate-one"
```
- 귀찮아서 bashrc에 등록해서 사용


## 답 확인
- {테스트이름}.ck 
- 이게 해당 테스트의 답지