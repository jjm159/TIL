# 8주차
- 핀토스!!!
- 동시성 제어 (뮤텍스, 세마포어, 모니터)

---

# 학습 키워드 정리

### CPU Scheduling 알고리즘
- 비선점형
    - FCFS (First Come First Served) / FIFO (First In First Out)
    - SJF
    - 우선순위
- 선점형
    - SRT (Shortest-Remaining time)
    - 라운드 로빈
        - time slice
    - MLFQ (Multi Level Feedback Queue)
- 현재 유명한 os들의 스케줄링 알고리즘
    - 거의 다 MLFQ다!

### Semaphore와 Mutex
- mutex는 공유 자원에 접근 가능한 스레드가 한 개 일 때만 사용 가능
- Semaphore는 변수 count 해서 공유 자원에 접근 가능한 스레드를 N개로 만들 수 있음
    - count가 1로 초기화되면, mutex랑 똑같음

### Race Condition (경쟁 조건)
- 임계 영역(Critical Section)
    - 여러 스레드가 동시에 접근했을 때 결과가 항상 같지 않은 영역
    - 그래서 이 영역은 지켜워야 함
        - 이게 `원자성`
- 근데 임계영역에 여러 개의 스레드가 접근하면 결과가 일정하지 않음
- 이걸 `race condition`이라고 함
- 이러한 임계 영역에 동시에 스레드들이 접근하지 못하도록 순서의 제어가 필요한데, 이것을 `동기화` 라고 함
- 이런 영역에 접근하는 여러 스레드들이 리소스 사용이 끝나기를 서로 기다리는 상태를 `데드락`이라고 함

### Deadlock
- 위에 참고

### Context Switching
- 프로세스 또는 스레드의 컨텍스트(상태)를 나중에 복원할 수 있도록 자신의 메타데이터(PCB, TCB)를 저장하고, 다른 프로세스 또는 스레드의 상태 메타 데이터를 로드해서 실행하는 것
- 프로세스의 context switching과 스레드의 그것은 다름
    - 스레드는 주소공간을 그대로 사용함
    - tcb - pc, 실행 중인 레지스터값들, stack pointer, 우선순위, 스레드 상태, tid
    - 주소공간의 교체 없이, tcb만 교체하면 되므로, 스위칭 비용이 프로세스보다 저렴
- 이 작업은 실제 프로그램의 작업과는 전혀 상관 없는 것이므로, 순수한 오버헤드임

### Multi-Level Feedback Queue Scheduler (MLFQS)
- 규칙
    - 1. 우선순위가 높은게 먼저 실행
    - 2. 우선순위가 같으면 RR 방식으로 실행
    - 3. 첫번째큐가 가장 높은 우선순위이고, 처음 시스템이 실행되면 모두 여기에서 시작
    - 4. time slice 규칙
        - a. time slice를 다 사용하면 내려감
        - b. time slice 다 쓰지 않고 양도하면 유지
- 문제와 해결
    - starvation
        - 영원히 실행되지 못할 수도 ..?
        - 일정 시간(부두상수, s)이 지나면 전부 첫번째 큐로 놓고 시작
    - 스케줄링 조작 문제
        - time slice를 다 쓰지 않고 반환하는 것을 반복해서 우선순위를 높게 유지하는 나쁜 프로세스 존재
        - 정해진 총 할당량을 소진하면 내림
- 부두상수 s는 어떻게 정할까?
    - 알아서 - os 개발자의 노하우
- 4BSD 알고리즘
    - 다단계 큐
    - 우선순위 조정
    - 타임 슬라이스
    - 선점 가능
- nice
    - 우선순위 조정 명령어
- 그렇다면 왜 mlfq를 쓰는가?
    - 프로세스가 언제 끝나는지 몰라서

---

## Project 0: PintOS

### Virtual Machine ~= Hypervisor
- qemu
    - cpu 시뮬레이터

### Common bugs
- Memory leak
    - free 안해줄 때 생기는 문제
- Race condition
    - 동시에 자원에 접근해서 발생하는 문제
- Deadlock (교착상태)
    - 서로 자원을 기다리는 상태 - 프로그램 진행이 안됨
    - 4가지 조건
        - 비선점 자원
        - 자원이 상호 배제 영역 이어야 
        - 보류 및 대기
        - 순환대기 - 서로 기다리는 상태
    - 해결 방법
        - 방법은 많지만, 보통 순환대기 조건을 방지함으로써 해결
        - 리소스 접근 순서를 정한다!
- Use after free
    - 할당 받은 메모리를 free한 후, 접근하는 문제
        - free를 또 하거나
        - 해당 주소로 데이터 접근을 시도하거나
    - 언제나 NULL 체크 해줘야 함
    - free 후에는 NULL을 할당해줘야 함

### Timer Interrupt
- time slice 소진했을 때 선점권을 빼앗아 다른 스레드를 실행시키기 위해, 실행중인 스레드에 interrupt를 날려서 switching을 실행

### Timer sleep
- 일정 시간동안 cpu 점유를 하지 않도록 함
- 그 시간동안 스케줄러의 대상에서 제외 됨
- 그 시간이 지나서 깨어나면 바로 스케줄링의 대상이 됨

---

# 퀴즈 

