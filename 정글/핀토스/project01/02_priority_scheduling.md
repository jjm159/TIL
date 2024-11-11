# Priority Scheduling

앞서, busy wating 방식의 sleep을 thread를 재우는 방식으로 변경함으로써 os에서 스레드를 어떻게 관리하는지, 인터럽트는 어떻게 이루어지는지에 대해 알 수 있었다.
이번에는 스케줄링 그 자체보다는, 스케줄링의 조건에 대해 집중해서, 우선순위에 따른 스케줄링을 직접 만들어보면서 그 과정을 이해해볼 것이다.

- [Priority Scheduling](https://casys-kaist.github.io/pintos-kaist/project1/priority_scheduling.html)

---

## 문제 

### priority inversion
- H, M, L 세 개의 스레드가 있다고 침
- 우선순위는 H > M > L 이다.
- H는 L이 가지고 있는 리소스에 의존하고 있다.
    - L이 가진 락을 기다리고 있는 상황! 
- H가 M보다 우선순위가 높아서 먼저 실행되어야 하지만,
- L이 M보다 우선순위가 낮아서 M이 먼저 실행되고,
- 그 동안 H는 실행되지 못하는 상태가 된다.
- M보다 우선순위가 높은 H 대신에 M이 실행되는 상태가 발생하는 것이다.
- 이런 상황을 `priority inversion` 이라고 한다.

### priority donation
- 그래서 H는 L에게 자신의 우선순위를 donation한다. 
- 그래서 적어도 H의 우선순위로 L이 실행되도록 해서 M이 H보다 먼저 실행되는 상황을 막는다.
- 이 `priority donation`을 구현하는 게 이번 과제의 목표다.

### different situations
- `multiple donations`
    - L보다 우선 순위가 더 높은 2개 이상의 스레드에서 L 스레드에 priority donation을 하는 상황
        -> 아마 더 높은 우선순위를 유지 시키면 되지 않을까?
- `nested donation`
    - H가 M이 가진 락을 기다리고, M이 L이 가진 락을 기다리는 상황에서,
    - H는 L까지 priority donation을 해야 실행된다.
    - 필요하다면, 깊이의 제한을 둘 수도 있다. 8 levels 이런식으로.

### 락만!
- 락에 대한 priority donation만 구현! 다른 동기화 상황에 대해서는 고려 x

### 구현해야 하는 거
- thread가 자신의 우선순위를 체크해서 조치를 취하도록 뭔가 처리가 필요하다.

```c
void thread_set_priority (int new_priority);
```
- 현재 스레드의 새로운 우선순위가 이전보다 낮아졌다면, scheduling을 다시 해줘야 한다.
- 지금 실행되면 안되는 스레드가 되어버린 것일 수도 있기 때문.


```c
int thread_get_priority (void);
```
- donation이 존재하면, 더 높은 우선순위를 가지는 스레드의 우선순위를 반환해야 한다.


### 하면 안되는 거
- 다른 스레드의 우선순위를 수정하는 짓을 하면 안된다.

### 앞으로 사용 X
- 이 우선 순위 스케줄러는 이후에 프로젝트에서 사용되지 않는다!

---

## 우선순위 스케줄링

### 지금 우선순위를 체크해서 스케줄링을 하고 있는가?
- 우선순위를 체크해서 가져오는 부분
- 체크를 하긴 하는지? 
- thread 구조체에 있는 priority 멤버 변수를 사용하는 곳을 찾아보았다. 우선순위를 설정해주고, 우선순위를 조회하는 함수를 제외하고는 사용하는 곳이 없었다.
- 충격적이게도, 지금 우선순위가 높은 아이가 먼저 실행되고 있지 않다!!!
- 이것 먼저 추가해주어야 한다.

### 어디를 수정해야 하는지? 
- ready_list에 애초에 priority 순으로 정렬해서 넣고, 항상 맨 앞에 있는 값을 가져오는 방식으로 구현
    - ready_list에 insert하는 곳을 찾아가서 ordered를 사용하도록 수정
    - ready_list에서 가져올 때 front에 있는 아이를 가져오는 지 확인 - 아니면 수정
- 다시 스케줄링하기
    - priority를 설정하는 곳에서 현재 실행 중인 스레드의 우선순위가 더 낮아지는 경우 스케줄링을 다시 해줘야 한다.
        - 스레드를 생성하는 경우
            - thread_create
        - 스레드의 우선순위를 다시 설정하는 경우
            - thread_set_priority

### cpu 점유권 내주기
- preemtion
    - 현재 설정된 스레드의 우선순위가 read_list에 있는 스레드들의 우선순위보다 낮다면, yield
    ```c
    void preemption(void)
    {
        if (
            !list_empty(&ready_list) 
            && thread_current()->priority < list_entry(list_front(&ready_list), struct thread, elem)->priority
        ) {
            thread_yield();
        }
    }
    ```

### 정렬하기
- semaphore에서 락을 기다리고 있는 waiters에 있는 스레드들도 정렬이 되어 있어야 한다. 
- 또한 조건 변수를 사용할 때에도 세마포어들이 정렬이 되어 있어야 하는데, 여기서 정렬 조건 함수는 세마 포어의 waiters에서 thread를 꺼내서 priority를 확인해야 한다.

---

## 도네이션

- nested donation 처리
    ```c
    static void
    donate (void)
    {
        struct thread *current = thread_current ();
        while (current->lock_for_waiting != NULL)
        {
            struct thread *holder = current->lock_for_waiting->holder;
            holder->priority = current->priority;
            current = holder;
        }
    }
    ```
    - 적어도 내가 가진 priority 만큼은 만들어줘서 무한 대기하는 상황은 방지한다.
    - lock_acquire에서 holder가 있을 때 호출

- multiple donation 처리 필요
    ```c
    void
    adjust_priority (void) 
    {
        struct thread *current = thread_current ();
        current->priority = current->init_priority;
        if (!list_empty (&current->donations)) {
            list_sort (&current->donations, greater_priority_thread_donation, NULL);
            struct thread *front = list_entry (list_front (&current->donations), struct thread, donation_elem);
            if (front->priority > current->priority) {
                current->priority = front->priority;
            }
        }
    }
    ```
    - lock_release와 thread_set_priority에서 호출
    - donations에 있는 multiple한 thread 중 가장 우선순위가 높은 thread의 priority가 현재 lock을 획득한 thread의 priority보다 높으면 donate 적용

---
