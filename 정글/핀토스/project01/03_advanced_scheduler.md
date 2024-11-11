# Advanced Scheduler

- mlfq 레츠고~

## 문제 분석 

### Advanced Scheduler 
- multilevel feedback queue scheduler를 구현!
- average response time을 줄이기 위해!
- advanced scheduler는 donation을 하지 않는다.
- 그래서! advanced scheduler를 사용해서 작업하기 전에, donation을 제외하고서라도 우선순위 큐가 동작하게 만드는 것을 추천! 
    - ?????????? 그럼 앞에꺼 왜ㅠ
- parse_options
    - mlfq 옵션을 줘서 분기를 탈 수 있게 만든다. 
    - thread_mlfqs를 보고 분기처리하면 된다.
    ```c
    #ifdef FILESYS
            else if (!strcmp (name, "-f"))
                format_filesys = true;
    #endif
            else if (!strcmp (name, "-rs"))
                random_init (atoi (value));
            else if (!strcmp (name, "-mlfqs"))
                thread_mlfqs = true;
    #ifdef USERPROG
    ```
- thread_mlfqs가 true면 get에서 init_priority를 반환해야 한다!
    - 굳이 donation 동작들을 건드릴 필요 없이 이것만 하면 될까?
- 암튼 advanced scheduler는 앞으로 프로젝트에서 사용하지 않을 것이라고 한다!

### 4BSD Scheduler
- 반응시간!!! 
    - I/O 작업이 많으면 - cpu 그닥, fast response time 필요
    - compute-bound 스레드는 많은 cpu time 필요, fast response time 필요 없
    - 이걸 스케줄러가 잘 반영하면, 모든 requirements에 대해 동시적으로 반응할 수 있다!
- 구현해야 하는 mlfq
    - 우선순위가 부여된 ready 큐가 여럿 있음
    - 비어 있지 않은 ready queue 중에 가장 우선순위가 높은 thread 우선 실행
    - 각 ready queue는 여러 스레드를 가지고 있고, 이 녀석들은 round robin 방식으로 실행됨
- timer_ticks
    - a certain number of timer ticks 이후에 업데이트가 되어야 한다! 
    - 스케줄러 업데이트 하는 동안, 또 tick interrupt가 발생되어 아직 업데이트가 안된 스케줄러의 데이터를 보지 않도록!
- priority donation
    - 하지 않는다!

### `Niceness`
- nice value
    - 스레드의 우선순위는 여러 공식에 의해 계산되는데, 또한 각각의 스레드는 정수형 데이터이 ㄴnice value를 가진다.
    - 얼마나 다른 스레드에게 친절한지를 나타낸다.
- 0
    - 영향 X
- +
    - 스레드의 우선순위를 낮춰서 양보한다.
    - 최대 +20
- -
    - 다른 스레드의 cpu 시간을 빼앗아버린다.
    - 최소 -20
- nice value 규칙
    - 초기에는 0
    - 자식 스레드는 부모 스레드로부터 물려받음
- 구현
    - int thread_get_nice(void)
        - 현재 nice value 반환
    - int thread_set_nice(int nice);
        - 현재 nice value 설정
        - 스레드 우선순위 다시 계산
            - 지금 실행중인 스레드의 우선순위가 highest가 아니면 yield

### Calculating `Priority`
- 기초
    - 64개의 priority를 가지고, 그렇다면 큐도 64개가 있다.
        - PRI_MIN
            - 0
        - PRI_MAX
            - 63
    - 각 큐는 0~63으로 numbering 된다.
    - 숫자가 클 수록 우선순위가 높다.
    - thread 초기화할 때, 우선순위를 처음 계산한다.
    - 4 clock tick이 지날 때마다 다시 계산한다.
    - 우선순위는 다음 공식으로 계산된다.
    ```
    priority = PRI_MAX - (recent_cpu / 4) - (nice * 2),
    ```
- 공식 설명
    - `recent_cpu`
        - cpu time의 추정값 that 스레드가 최근에 사용한!
    - `nice`
        - thread가 가지고 있는 nice value
    - result
        - 결과 값은 가장 가까운 정수 값으로 절삭!
    - 1/4, 1/2
        - 는 믿고 쓰면 됨!
- 공식 효과
    - cpu를 최근에 사용한 스레드에가 낮은 우선순위를 준다.
    - 이게 starvation을 방지해준다.

### Calculating `recent_cpu`
- 각 프로세스가 최근데 얼마나 cpu time을 받았는지 측정한 갑싱 recent_cpu
- 최근 cpu time이 더 heavily하게 weighted되어야 함
- 최근 n초 동안 받은 cpu time을 track하기 위한 array를 사용해서 구현할 수도
    - 하지만 스레드마다 O(N)의 공간이 필요
    - 새로운 weighted average를 계산할 때마다 O(N) 시간 필요
- exponentially weighted moving average
    ```
    x(0) = f(0)
    x(t) = ax(t-1) + (1-a)f(t)
    a = k/(k+1)
    ```
- x(t)
    - t>=0일 때 정수 시간에서의 moving average
- f(t)
    - averaged가 되는 함수
- k > 0 
    - decay를 제어


### Calculating `load_avg`
- 지난 1분동안 실행 준비가 된 스레드의 평균 수 추정
- 스레드에 특정되지 않고 시스템 전체
- 1초마다 공식대로 업데이트
```
load_avg = (59/60) * load_avg + (1/60) * ready_threads
```
- ready_threads
    - 업데이트 시점에 실행중이거나 실행 준비가 된 스레드 수
    - 유휴 스레드 포함 X
- timer_tikcs() % TIMER_FREQ == 0 일 때만 업데이트

### Summary

### Fixed-Point Real Arithmetic
- 요게 관건..!
- 부동 소수점 실수 연산은 커널을 느리게 만들고 구현이 복잡하다.
- 그래서 정수를 사용한 고정 소수점 연산 방식을 사용한다.
- 위 공식들을 고정 소수점 연산으로 계산하는게 이번 과제의 핵심

---
