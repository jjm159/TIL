# P2 Introduction

## User Programs
- user programs를 돌리는 게 목표!
- 지금은, 돌릴 수는 있지만 I/O도 없고 상호작용하는게 불가능하다.
- system call을 사용해서 이걸 가능하게 하고 user program을 실행 가능하도록 할 것!
- `userprog` 디렉토리!


## Background

- 권한
    - 지금까지 모든 코드는 커널단에서 동작되었다.
    - 커널의 부분을 만든 것!
    - 그래서 풀 엑세스 권한을 받았다.
    - 이제부터는 유저 프로그램을 돌릴거고, 그러면 풀 엑세스 권한을 받는건 더이상 아닐 것
- 멀티 프로세스
    - 한 번에 여러 프로세스 실행 가능(멀티스레드 프로세스는 지원되지 않음)
    - 각 프로세스는 하나의 스레드를 가짐 
    - **유저 프로그램은 컴퓨터 전체를 사용한다는 illusion을 가진 상태에서 만들어진다.**
    - 그래서, 여러 프로세스를 한번에 로드하고 실행할 때, illusion을 유지하기 하기 위해 메모리, 스케줄링, 서로 다른 상태들에 대해 관리가 필요하다.
- 유저 프로그램
    - 지금까지는 커널에서 돌아가는 코드였고, 이게 실행되려면 커널에서 호출할 인터페이스 함수가 필요했다.
    - 이제부터는, 이런 `직접 만든 코드 인터페이스`가 아니라, `유저 프로그램을 실행`하여 테스트할 것이다.
    - 코드는 #ifdef VM 안에 쓰면 안된다. 이건 project3에서 가상 메모리 시스템을 구현한 후에 포함될 것
- 읽어봐
    - synchronization
    - virtual address

## Source Files
- process.c, process.h
    - ELF 바이너리들을 로드하고 프로세스들을 시작해준다.
- `syscall.c`, `syscall.h`
    - 유저 프로세스가 커널 기능에 접근할 때에는 시스템 콜이 필요하다.
    - 지금은 system call 핸들러의 골격. 메시지를 출력하고 user process를 종료시킴
    - project2는 이 골격 안에 코드를 채워 넣는 것
- syscall-entry.S (수정 X)
    - 이해할 필요 X. system call handler를 초기화 해줌
- `exception.c`, `exception.h`
    - 유저 프로세스가 특권이 필요하거나 금지된 작업을 수행하면, 유저 프로세스는 커널로 빠져든다. exception이나 fault 같은 걸로!
    - 이 파일들은 exceptions를 다룸
    - 현재 모든 예외들은 간단히 메시지를 출력. 프로세스를 종료함.
    - project 2에서는 `page_fault()`만 할 것
- gdt.c, gdt.h (수정 X)
    - x86-64는 segmented 아키텍쳐다.
    - Global Descriptor Table(GDT)는 사용중인(in use) segments를 적어 놓은 테이블이다.
    - 이 파일들은 GDT를 구성한다.
    - 이 파일은 지금도, 앞으로도 수정할 일 없다.
- tss.c, tss.h (수정 X)
    - Task-State Segment (TSS)는 x86 아키텍처의 task 전환에 사용된다.
    - TSS는 지금 사용되지 않지만, 링 스위칭 동안 stack pointer를 찾기 위해 여전히 존재함
    - user process가 interrupt handler에 진입할 때 하드웨어는 TSS를 참조해서 커널의 스택포인터를 확인.
    - 수정 안할 것!

## Using the File System
- 파일 시스템 코드의 인터페이스들을 많이 사용하게 된다. 유저 프로그램은 파일 시스템에서 로드되고, 앞으로 구현할 시스템 콜들에서 파일 시스템을 사용하게 된다.
- 하지만 이 코드들을 수정할 일은 없을 것. project 4나 가야 수정할 것 
- 수정하기 전까지 다음과 같은 `limitations`들을 좀 참아야 함 
    - 파일 시스템 내부적으로 동기화 처리 X, 파일시스템 호출하는 쪽에서 동시성 문제 해결해줘야 해줘야..!
    - 파일 크기가 생성 시간에 결정됨. 루트 디렉터리는 파일로 표현. 생성 파일 수 제한.
    - 파일 데이터는 단일 범위로 할당. 단일 파일 데이터는 디스크 내에서 연속 섹터 범위에 잇어야 함. 외부 단편화 심각해질 수 있음
    - 하위 디렉터리가 없음
    - 파일 이름은 14자 제한
    - 작업 도중 시스템이 중단되면 디스크 자동 복구 안됨. 손상될 수 있음.
- 중요한 기능
    - `filesys_remove()`
    - 유닉스 스타일의 파일 삭제 규칙 구현
    - 파일이 삭제될 때 열려 있는 상태면, 블록 해제 X, 열려 있는 스레드들이 파일을 닫을 때까지 접근 가능
- 프로젝트 1과 다른 점
    - 커널 이미지에 테스트 프로그램 포함 X
    - 테스트 프로그램이 Pintos 가상 머신에 있어야 함
    - 대부분 make check 같은 스크립트에서 처리해줌
    - 파일을 pintos 가상 머신에 추가하려면 시뮬레이션 디스크 생성 필요
        - `pintos-mkdisk`
        - userprog/build 에서 다음 명령어를 실행
        ```
        pintos-mkdisk filesys.dsk 2
        ```
        - filesys.dsk라는 이름의 2MB 핀토스 파일 시스템 파티션을 포함한 시뮬레이션 디스크 생성
        - 디스크 지정을 위해 다음 명령어 추가 실행
        ```
        pintos --fs-disk filesys.disk -- KERNEL_COMMANDS...
        ```
        - --은 -fs-disk가 Pintos 스크립트용이고 시뮬레이션된 커널의 것이 아님을 의미
        - -f, -q를 추가해서 파일 시스템 파티션을 포맷
            - -f: 파일 시스템 포맷
            - -q: 포맷 완료시 Pintos 종료
    - 가상 시스템에서 파일 복사 
        ```
        pintos -p file -- -q
        ```
    - 다른 이름으로 복사
        ```
        pintos -p file:newname -- -q
        ```
    - 위 명령어에서 q를 g로 바꾸면 VM에서 파일을 가져옴
- 명령어 최종 정리
    ```
    pintos-mkdisk filesys.dsk 10
    pintos --fs-disk filesys.dsk -p tests/userprog/args-single:args-single -- -q -f run 'args-single onearg'
    ```
- 파일 시스템 디스크 유지를 하기 싫으면, -filesys-size=n 옵션 추가. 실행 동안만 nMB 크기의 임시 파일 시스템 파티션 생성
    ```
    pintos --fs-disk=10 -p tests/userprog/args-single:args-single -- -q -f run 'args-single onearg'
    ```

## How User Programs Work
- 메모리에 맞고, 구현한 시스템 호출만 사용하는 C 프로그램을 실행할 수 있음
    - malloc은 구현 불가 
    - 부동소수점 연산 프로그램 구현 불가
- userprog/process.c의 load로 ELF 실행 파일 로드 가능
- x86-64 ELF 실행 파일 생성 가능한 컴파일러의 링커로 Pintos용 프로그램 생성 가능

## Virtual Memory Layout
- Pintos의 가상 메모리는 사용자 가상 메모리와 커널 가상 메모리 두 개 영역으로 나누어짐
    - 사용자 가상 메모리 영역
        - include/threads/vaddr.h의 KERN_BASE까지 범위 가짐
        - 기본값 0x8004000000
    - 커널 가상 메모리
        - 나머지 가상 주소 공간
- 사용자 가상 메모리는 프로세스마다 다름
    - 프로세스 전환시 커널은 프로세서의 페이지 디렉터리 베이스 레지스터를 변경해서 사용자 가상 주소 공간을 전환함
    - thread/mmu.c의 pml4_activate() 참조
    - struct thread에 프로세스 페이지 테이블에 대한 포인터 포함
- 커널 가상 메모리는 전역적
    - 어떤 프로세스나 스레드가 오든 항상 동일하게 매핑
    - 핀토스에서 커널 가상 메모리
        - KERN_BASE 부터 실제 메모리(물리 메모리)와 일대일 매핑
- 사용자 프로그램은 자신의 가상 메모리에만 접근 가능
    - 커널 가상 메모리 접근시 page fault
    - userprog/exception.c에서 page_fault()에서 처리, 프로세스 종료
- 커널 스레드는 자신의 가상 메모리와 유저 프로그램의 가상 메모리 모두 접근 가능
    - 커널도 매핑되지 않은 사용자 가상 주소 접근시 페이지 폴트

## Typical Memory Layout
- 사용자 스택의 크기는 고정되어 있음
- project 3에서 가변적으로 확장될 수도 있음
- 초기화 되지 않은 데이터 세그먼트의 크기는 시스템 호출로 조정 가능. 여기서 구현 X
- code segment는 0x400000에서 시작
    - 주소 공간의 하단에서 128mb 떨어진 위치
- user program의 메모리 레이아웃은 링커 스크립트로 지정
    - 다양한 세그먼트의 이름과 위치를 지정
- objdump -p 로 실행 파일의 메모리 레이아웃 확인 가능

## Accessing User Memory
- 커널은 사용자 프로그램이 제공한 포인터로 메모리에 접근하는 경우가 많은데, null 포인터이거나 매핑 안된 가상 메모리의 포인터 등 잘못될 수 있으니 주의 필요
- 예외 처리 해줘야 함 
- 올바르게 동작하게 만드는 두 가지 방법
    - 포인터 유효성 체크 
        - thread/mmu.c, include/threads/vaddr.h 함수 참조
    - KERN_BASE 아래를 가리키는지 확인
        - userprog/exception.c의 page_fault() 수정
- 어떤 방법을 사용하더라도 누수 않하게 조심
    - 첫번째 방법은 간단하게 처리 가능
    - 페이지 폴트 시 오류 코드 반환 방법이 없음