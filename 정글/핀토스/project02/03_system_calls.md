# System Calls
- [깃북](https://casys-kaist.github.io/pintos-kaist/project2/system_call.html)

---
## 문제 이해하기
- userprog/syscall.c 에 있는 시스템 콜 구현

### 시스템 콜 구현하기
- userprog/syscall.c에 있는 system call handler 구현
- 시스템 콜 호출 번호를 찾아서 필요한 argument를 가져와 해당되는 작업을 처리

### system call details
- 첫 프로젝트에서 했던 타이머 인터럽트는 `외부 인터럽트`. 외부 장치에 의해 발생했기 때문
- 프로그램 내에서 발생하는 예외도 있는데 이를 `내부 인터럽트`라 하고, 0으로 나누기 등의 에러가 그 예다.
- systemcall은 이러한 예외를 통해 실행된다.
- x86에서는 `syscall` 이라는 명령어를 사용해서 이러한 인터럽트를 발생시켜서 시스템 콜 핸들러를 호출한다.
- 시스템 호출 번호와 인수는 syscall 호출 전에 레지스터에 세팅된다 (어셈블리 코드)
    - %rax에 시스템 호출 번호 
    - 네번째 인수는 %rcx가 아니라 %r10에 할당
- syscall_handler가 제어권을 얻게 된다면, rax에는 시스템 호출 번호가, rdi, rsi, rdx, r10, r8, r9 순서로 인수가 있을 것
- 해당 레지스터는 struct intr_frame에 있음
- 함수 반환값은 rax를 수정해서 전달
-> 이제 시스템 콜 각각을 구현하면 된다.

---
## 구조 이해하기
###  `syscall_handler`가 어디서 어떻게 호출되는 걸까?
- 바로 전체 검색!
- syscall_entry
    ```
    check_intr:
        btsq $9, %r11          /* Check whether we recover the interrupt */
        jnb no_sti
        sti                    /* restore interrupt */
    no_sti:
        movabs $syscall_handler, %r12
        call *%r12
    ```
- `syscall-entry.S` 어셈블리 코드에서 호출된다.

### 그럼 syscall_entry는 어디서 호출되는가?
- syscall_init에서 syscall_init를 msr에 등록
```c
void
syscall_init (void) {
    // ...
	write_msr(MSR_LSTAR, (uint64_t) syscall_entry);
    // ...
```
- msr은 Model-Specific Register의 약자로, x86 및 x86-64 아키텍처에서 특정 CPU 모델에 따라 제공되는 특수 레지스터
- 인터럽트 같은 동작을 제어하기 위해 cpu에 해당 주소를 등록해줘야 하는데, 이 주소를 등록하는 레지스터가 cpu 모델에 의존적
- 이 함수는 x86 아키텍처의 시스템 콜 레지스터 등록 프로토콜에 따라 syscall_entry 함수 포인터를 해당 레지스터에 등록하는 과정
- 시스템 콜은 이렇게 entry를 os가 등록해주고, cpu가 특정 시스템 콜 호출시 이 주소로 syscall_entry를 실행하게 됨

### 인터럽트랑은 다른 것인가?
- 전통적인 방식으로는 인터럽트 벡터에 syscall을 등록해서 사용
- x86-64에서는 더 효율적인 system call 메커니즘인 syscall/sysret 명령어 도입
- 인터럽트 벡터를 사용하지 않고, MSR을 사용해서 시스템콜 핸들러를 직접 지정
- 장점
    - 기존 인터럽트 벡터를 사용하는 방식보다 명령어 사이클이 적게 소요
    - 소프트웨어 인터럽트를 사용하지 않아, 시스템 콜 처리와 하드웨어 인터럽트 처리가 명확히 분리

### user의 system call과 핀토스의 system call은 다른 것!
- 우리가 구현하는건 user 프로그램의 system call
- 유저 프로그램이 system call을 호출했을 때 핀토스 커널 스레드의 syscall_handler가 실행되고, 
- 여기서 분기를 쳐서 각각의 시스템 콜을 구현해주면 된다!

---

## 기타 추가 사항
- Introduction에서 Using the File System에 아직 동기화가 제대로 이루어지지 않았다는 내용에 따라 system call 락처리 필요
- 초기화
    ```c
    struct lock file_sys_lock;

    void
    syscall_init (void) {
        // ...
        lock_init(&filesys_lock);
    }
    ```