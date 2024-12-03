# Stack Growth
- 스택이 크기를 초과하면 페이지를 추가로 할당
- 페이지 폴트 중에서도 스택 크기 초과인 경우에만 추가 페이지 할당
- 문제
    - stack pointer 아래로 접근 시 버그
    - x86-64에서 PUSH 명령어는 sp보다 8바이트 아래에서 페이지 폴트 발생 가능
- 해결
    - rsp를 syscall_handler() 또는 page_fault()에서 struct intr_frame의 rsp 멤버를 사용
    - 페이지 폴트가 커널에서 발생시, rsp 값이 정의 되지 않을 수 있음
        - 사용자에서 커널로 전환시 rsp를 struct thread에 저장 필요

## Implement stack growth functionalities
- vm/vm.c의 vm_try_handle_fault를 수정
- 스택 증가 식별
- vm/vm.c의 vm_stack_growth를 호출하여 스택 확장

#### vm_try_handle_fault
```c
bool vm_try_handle_fault (struct intr_frame *f, void *addr, bool user, bool write, bool not_present);
```
- page_fault(userprog/exception.c)에서 페이지 폴트 예외를 처리하는 동안 호출
- 페이지 폴트가 스택 증가를 위한 유효한 케이스인지 확인
- 페이지 폴트가 스택 증가로 처리될 수 있다고 확인되면, 해당 주소를 사용해 vm_stack_growth를 호출

#### vm_stack_growth
```c
void vm_stack_growth (void *addr);
```
- 스택 크기를 증가시키고, 익명 페이지(anonymous pages)를 하나 이상 할당
- addr이 더 이상 페이지 폴트를 발생시키지 않도록
- 페이지를 할당할 때 addr을 PGSIZE로 내림하여 정렬

#### 추가
- 대부분 운영체제는 스택 크기에 절대적 제한을 둠
- 리눅스는 보통 - 8MB
- 이번 프로젝트에서는 1MB로 제한