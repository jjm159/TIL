# Argument Passing 

- 함수 호출 규약에 맞게 argument를 넣어주는 작업
- 스택 포인터는 어디에?
- 여기서 어떻게 데이터를 할당하면 되나?

---

## 문제

- `process_exec()` 완성하기!

### x86-64 Calling Convention
- 함수 호출 규약
    - 유저 레벨 앱 - rdi, ..., 등의 레지스터 사용
    - caller는 callee가 return한 뒤에 실행할 instruction의 주소를 stack에 push
    - caller는 callee의 first instruction으로 jump
        - `CALL` 이라는 명령임
    - callee 작업 완료
    - callee는 return value가 있다면 `RAX`에 저장
    - stack을 pop해서 caller가 저장해 놓은 명령어의 주소로 jump
        - `RET` 명령어

### Program Startup Details
- `_start()`
    - lib/user/entry.c
    - 유저 프로그램의 시작 점 
    - main을 호출하고, main이 return하면 exit 실행
- 커널은 프로그램이 실행하기 전에 초기 함수의 arguments를 레지스터에 넣어야 함
- 예시 
    - `/bin/ls -l foo bar` 명령어 실행
    - /bin/ls , -l , foo , bar 로 쪼갬
    - stack top에 위치. 순서 상관 없음
    - 각 string의 address와 null pointer sentinel을 스택 위에 오->왼 순으로 push
    - 이 값들은 `argv`임
    - argv[argc]가 null pointer sentinel 이 되어야 함 (표준)
    - `rsi`에는 argv의 주소(&argv[0]), `rdi`에는 argc
    - 마지막에는 return value가 없어도 fake return address를 push
        - `rax` 에 저장

### Implement the argument passing.

- process_exec() 채우기
- program file name을 공백 기준 짜름
- 여러 연속된 공백은 하나의 공백으로 처리
- strtok_r() 사용

---

## 진입점부터 살펴보기
- process_create_initd 실행
    - 여기서 thread_create를 하는데, 실행할 작업에 initd를 넣음
- initd
    - 여기서 process_exec가 실행된다!
- process_exec가에서 load를 실행
- load에서 setup_stack하고 그 바로 아래에 args 세팅을 해주면 된다! 

## load에서 파일 로드 에러
```c
static bool
load (const char *file_name, struct intr_frame *if_) 
```
- file_name에 명령어와 인자들이 함께 들어온다.
- 문자열을 공백을 기준으로 분리해서 file_name에는 파일 이름만 들어가도록 해야 한다. 그렇지 않으면 오류 발생.

## 스트링 분리하는 로직
```c
    char *ptr, *arg;
    int argc = 0;
    char *argv[64];

    for (arg = strtok_r(file_name, " ", &ptr); arg != NULL; arg = strtok_r(NULL, " ", &ptr))
        argv[argc++] = arg;
```
- strtok_r 내부에서 string인자에 NULL을 넣어주면, 이전 토큰 다음 위치인 ptr을 현재 string으로 할당해주어서 연속적으로 토큰을 순회할 수 있도록 해준다.

## load 함수의 역할
- 운영체제 마다 실행 가능한 파일의 포맷이 정해져 있다. 
- 그래서 디스크에서 가져온 비트열의 특정 주소에 약속된 값이 적혀 있어야 한다.
- load 함수에서는, 약속된 위치에서 해당 데이터들을 읽어서 프로세스의 주소 공간을 세팅해준다.
- 그리고 intr_frame에 프로그램의 시작 주소와, stack의 시작주소를 세팅해준다.

## argv 스택에 세팅하기
- load에서는 실행 인자로 받은 데이터들을 함수 스택에 세팅해주는 역할도 함께 수행한다.
- 이게 구현해야 하는 내용이고, 구현 결과는 다음과 같다.
```c
static void argument_stack(char **argv, int argc, struct intr_frame *if_) 
{
    char *arg_addr[100];
    int argv_len;

    // argv의 인자들을 역순으로 스택에 저장 (right to left) 
    for (int i = argc - 1; i >= 0; i--) {
        argv_len = strlen(argv[i]) + 1;
        if_->rsp -= argv_len;
        memcpy(if_->rsp, argv[i], argv_len);
        arg_addr[i] = if_->rsp;
    }

    // rsp가 8의 배수가 되도록 0으로 패딩을 추가
    while (if_->rsp % 8)
        *(uint8_t *)(--if_->rsp) = 0;

    // argv[argc]에 NULL 포인터 설정
    if_->rsp -= 8;
    memset(if_->rsp, 0, sizeof(char *));

    // argv 각 인자의 주소를 역순으로 저장
    for (int i = argc - 1; i >= 0; i--) {
        if_->rsp -= 8;
        memcpy(if_->rsp, &arg_addr[i], sizeof(char *));
    }

    // return할 address 설정
    if_->rsp = if_->rsp - 8;
    memset(if_->rsp, 0, sizeof(void *));
    if_->R.rdi = argc;
    if_->R.rsi = if_->rsp + 8;
}
```
