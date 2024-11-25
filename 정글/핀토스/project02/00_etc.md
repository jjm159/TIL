# ETC

## 트러블 슈팅

### OS 버전 문제 
- 드디어 우분투 18.04가 아닌 경우 문제 확인
- `make check` 실행시, test name을 발견할 수 없다는 에러와 함께 중간에 test가 종료됨

### 인터럽트 에러 문제
- 아무것도 작업하지 않고 userprog에서 make check 진행시, 지난 프로젝트 테스트 케이스에서 Fail
- 다른 팀에서는 Pass
- cpu 점유권을 내어줄 때 thread_yield를 호출하는데, 여기서 인터럽트 context이면 안되는데, 인터럽트 context에서 실행되어 터지는 문제
- 인터럽트 context일 때 예외 추가하여 해결
    ```c
    if (!intr_context ()) {
        thread_yield();
    }
    ```
    
### 명령어 꿀 빨기
- .bashrc.sh에 아래 것들 추가하고 `source .bashrc.sh` 실행
```bash
source /root/pintos-kaist/activate

alias p_make="cd /root/pintos-kaist/userprog;make clean;make;cd build"
alias p_args_single="cd /root/pintos-kaist/userprog/build;pintos --fs-disk=10 -p tests/userprog/args-single:args-single -- -q -f run 'args-single onearg'"
alias p_args_none="cd /root/pintos-kaist/userprog/build;pintos --fs-disk=10 -p tests/userprog/args-none:args-none -- -q -f run 'args-none'"
```
- p_make 하면 빌드, p_args_single 하면 저거 실행
- 두 개 동시에 하고 싶으면 p_make;p_args_single 이라고 치면 꿀

### print 확인하기
- print 확인하고 싶으면, 아래 코드 처럼 process_wait에 thread_sleep을 추가해줘야 함
- 이건 그냥 확인용!
```
int
process_wait (tid_t child_tid UNUSED) {
	/* XXX: Hint) The pintos exit if process_wait (initd), we recommend you
	 * XXX:       to add infinite loop here before
	 * XXX:       implementing the process_wait. */

	thread_sleep (1000);
	return -1;
}
```

### CommandLine Parsing을 구현할 때 주의할 점
process_exec 함수에서 보면

success = load(file_name, &_if);
palloc_free_page(file_name);
if (!success)
    return -1;
과 같은 방식으로 되어있는데,

만약 CommandLine Parsing을 load 함수 내에 구현하지 않는다면
반드시 palloc_free_page를 조건문으로 분기시켜 **정상적으로 load가 되었을 경우에는 CommandLine Parsing이 이루어진 이후에 실행되게끔 해야 한다.**
즉
success = load(file_name, &_if);

if (!success)
{
    palloc_free_page(file_name);
    return -1;
}
argument_stack(argv, argc, &_if);
//디버그용
hex_dump(_if.rsp, _if.rsp, USER_STACK - _if.rsp, true);
palloc_free_page(file_name);
과 같은 방식으로 해야되는 것이다.

### 테스트 케이스 돌릴 때 주의할 점
1. pintos --fs-disk=10 -p tests/userprog/halt -- -q -f run 'halt' (X)
2. pintos --fs-disk=10 -p tests/userprog/halt:halt -- -q -f run 'halt' (O)

1번은 디스크에 halt라는 이름으로 파일을 올리지 않았기 때문에, run 할때 찾을 수 없어서 Kernel panic이 생김

### 테스트 케이스 돌리는 법
pintos --fs-disk=10 -p tests/userprog/exec-once:exec-once -- -q -f run 'exec-once'
pintos -v -k -T 60 -m 20   --fs-disk=10 -p tests/userprog/fork-read:fork-read -p ../../tests/userprog/sample.txt:sample.txt -- -q   -f run fork-read


FAIL tests/userprog/no-vm/multi-oom
pintos -v -k -T 600 -m 20 -m 20   --fs-disk=10 -p tests/userprog/no-vm/multi-oom:multi-oom -- -q   -f run multi-oom