# Alarm Clock

대망의 핀토스. 과연 OS는 내부적으로 어떻게 생겼을까? 두근 두근

## 문제 
- [Alarm Clock](https://casys-kaist.github.io/pintos-kaist/project1/alarm_clock.html)

- 현재 timer_sleep 라는 함수의 코드를 보면 

```c
/* Suspends execution for approximately TICKS timer ticks. */
void
timer_sleep (int64_t ticks) {
	int64_t start = timer_ticks ();

	ASSERT (intr_get_level () == INTR_ON);
	while (timer_elapsed (start) < ticks)
		thread_yield ();
}
```
- 이렇게 while문으로 시간을 계속 체크하고 있다. 
- tick은 운영체제가 시간의 경과를 인식하는 최소 단위다. sleep에 주어진 ticks 만큼 시간이 지났는지 while문에서 계속해서 체크하고 있다.
- sleep을 실행하고 있는 스레드가, sleep을 하는 시간동안 정말 아무 것도 하지 않음에도 cpu를 계속해서 점유하고 있게 된다.
- 이러한 상황을 `busy wait` 라고 한다. 
- 스레드가 cpu의 점유권을 바로 내어주고, 일정 시간이 지나면 다시 시작되도록 구현하는게 이번 과제의 목표다. 

## 진입점부터 뜯어 보기 

### init.c

#### int main(void)
- main 함수가 pintos의 진입점이다.
- main에서는 os의 구성요소들을 init해준다.
	- thread, console, page, interrupt, keyboard 등 os가 관리하는 요소들을 초기화해준다.
- 이후, command line의 명령어를 파싱해서 argv에 담아 `run_actions`가 실행되는데, 이 부분이 테스트 코드들이 실행되는 곳이다. 
- USERPROG, FILESYS, VM과 같은 flag들이 define되어 있을 때 실행되는 로직들이 있는데, Project 1의 makefile에서 해당 flag들을 설정해주지 않아 아직은 고려하지 않아도 된다.
- 또한, run_actions를 하기 위해 `thread_start`을 통해 초기 idle 스레드를 설정해준다.
	- 여기서 idle 스레드는 초기 스레드로, 실행될 스레드가 없을 때 cpu를 좀 더 효율적으로 놀게 만들어주는 스레드다.

#### static void run_actions(char **argv)
- 마찬가지로 flag가 활성화되지 않아, actions에는 `run_task`만 초기화되고 실행된다.
- run_tesk에서는 다른 옵션이 활성화되지 않아, `run_test`가 실행된다.
- run_test에서 `tests`가 실행되는데, tests에는 테스트를 위해 정의한 함수들이 있고, 이 함수들을 for문을 돌면서 모두 실행하게 된다. 이제 이 함수들을 가서 확인해보면 된다.


## 문제 풀기 

- make check 또는 pintos -- run alarm-multiple > logfile 같은 명령어로 테스트를 돌려볼 수 있다. 
	- make check
	- pintos -- run alarm-multiple > logfile
	- pintos -T 10 -- -q run alarm-multiple
- 먼저 alarm-priority 이 녀석이 실패를 하는데, busy wait 때문에 실패했을 것이라 추정하고 해당 테스트 함수를 보았다.

### alarm-priority.c

#### test_alarm_priority
- 이 함수가 tests 배열에 담겨서 실행이 된다.
- 스레드를 10개를 생성하고, 각 스레드에서는 `alarm_priority_thread`가 실행되도록 한다.

#### alarm_priority_thread
- timer_sleep이 실행되는데, 이 녀석의 로직을 보면 busy wait를 하고 있는 것을 확인할 수 있다.

## 스레드 실행 로직 뜯어보기

- 스레드를 구현하게 된다면, 어떤 것을 먼저 생각하고 구현해야 할까? 
- 스레드의 데이터 구조, 여러 개의 스레드가 실행되는 지점, `선점`이 일어나는, 즉 context switching이 일어나는 지점, ..
- 우선 이렇게 떠오른다. 해당 작업들이 이뤄지는 곳을 thread.c에서 찾아보기로 했다.

### thread.c

#### '선점'이 이뤄지는 곳은 어디인가? 
- 이미 실행중인 스레드가 실행 시간을 모두 사용하면, yield를 통해 cpu 점유권을 내놓아야 하는데, 어디인가
- void thread_tick(void)
```c 
	if (++thread_ticks >= TIME_SLICE)
		intr_yield_on_return ();
```
- 여기에 time slice를 체크해서 넘어가면 intr_yield_on_return로 `yield_on_return`를 true로 만듬
- intr_handler에서 yield_on_return가 true면 thread_yield 실행
- thread_yield > do_schedule > schedule > thread_launch
	- `thread_launch`가 컨텍스트 스위칭 일어나는 곳
	- thread의 tf(intr_frame)를 가져와서 메모리에 세팅해준다
	- `do_iret`가 실행되는데, 이게 switch을 해주는 작업
	- 이를 통해 next thread의 context로 cpu가 세팅되면, 다음에 cpu가 연산하는 내용이 자연스레 이 thread의 context가 됨
	- 이게 스레드의 컨텍스트 스위칭!

#### 타이머 인터럽트에 timer_interrupt 등록, 실행
- 스레드의 컨텍스트 전환은 thread_tick에서 결정됨. 시간마다 timeslice 확인해서 결정
- `thread_tick`는 `time.c`의 `timer_interrupt`에서 실행된다.
- timer_interrupt는 timer_init에서 인터럽트 레지스터에 등록된다.
- 등록된 인터럽트는 intr_handler에서 실행된다.
- intr_handler는 어셈블리에서 실행해준다. 	-> 어디서 언제 실행되는지????? 
- intr-stubs.S 에서 cpu 레지스터에 등록
	- cpu 레지스터에 인터럽트 핸들러들을 저장하는 공간이 있다
	- cpu가 어떤 인터럽트에 어떤 번호를 부여하는지에 따라 os에서 인터럽트 핸들러 초기화 코드가 달라진다
	- timer는 0x20에 등록됨
- 이렇게 등록된 인터럽트 핸들러는 타이머 하드웨어가 신호를 보냈을 때 실행된다

#### 타이머의 주기는?
- timer_init에서 설정해주는 로직

```c
void
timer_init (void) {
	
	uint16_t count = (1193180 + TIMER_FREQ / 2) / TIMER_FREQ;

	outb (0x43, 0x34);    /* CW: counter 0, LSB then MSB, mode 2, binary. */
	outb (0x40, count & 0xff);
	outb (0x40, count >> 8);

	intr_register_ext (0x20, timer_interrupt, "8254 Timer");
}
```

- TIMER_FREQ가 타이머의 주기인데, 1초에 TIMER_FREQ만큼 실행된다.
- 현재 TIMER_FREQ는 100으로 설정되어 있다.
- 1초에 100번 인터럽트가 실행되는 것! 

#### 정리

- thread_yield가 새로운 스레드를 running 상태로 만들어준다.(컨텍스트 스위칭)
	- thread_yield > do_schedule > schedule > thread_launch > do_iret
	- thread_launch, do_iret가 새로 실행될 thread의 frame 값들을 레지스터에 세팅
	- cpu는 무지성으로 읽기만 하고, thread의 값들을 적절하게 바꿔주는 것이 컨텍스트 스위칭이다!!!
- 라운드로빈(time slice 소진)에 의한 컨텍스트 스위칭
	- 타이머 등록 
		- timer_init > intr_register_ext > register_handler (timer_interrupt 등록)
	- 인터럽트 실행
		- timer_interrupt > thread_tick > intr_yield_on_return
	- 인터럽트 핸들러에서 스레드 실행
		- intr_handler > thread_yield (intr_yield_on_return == true면)

- 처음에 생각했던 것과 다름
	- 처음에는 thread.start 같은게 있을 줄 알았는데, cpu 값을 세팅하는 게 다였음

## 그럼 해야하는게 뭐야?
- busy wait으로 thread를 계속 점유하고 있어서, sleep의 시간이 다 될 때 까지 time_slice를 모두 소진하게 됨
- 아애 cpu 점유를 하지 않도록 만들어야 함
- ready list에서 빼서 따로 sleep list에 보관 
- tick 마다 확인해서 sleep list에 뺄만한 녀석이 있는지 확인하고 있으면 빼서 ready list에 추가
- sleep list에는 시간 순으로 정렬이 되어 있어야 함
	- tick 마다 sleep list의 모든 값들을 비교하면서 가장 작은 값이 현재 ready에 추가되어도 되는지 확인하는게 비효율
	- 애초에 sleep list가 정렬되어 있으면 tick 마다 한 번만 확인하면 됨

## Try~~!!!

#### busy wait 철거

- 철거
	- while문을 제거하고 thread_sleep을 호출
	```c
	void
	timer_sleep (int64_t ticks) {
		int64_t start = timer_ticks ();
		int64_t awake_time = start + ticks;

		ASSERT (intr_get_level () == INTR_ON);

		thread_sleep(awake_time);
	}
	```

- thread_sleep
	```c
	void 
	thread_sleep(int64_t awake_ticks)
	{	
		struct thread *curr = thread_current ();

		enum intr_level old_level;

		ASSERT (!intr_context ());

		old_level = intr_disable ();

		curr->awake_ticks = awake_ticks;
		list_insert_ordered (&sleep_list, &curr->elem, less_awake_time_thread, NULL);

		if (curr != idle_thread) {
			thread_block ();
		}

		intr_set_level (old_level);
	}

	bool less_awake_time_thread(const struct list_elem *a,
								const struct list_elem *b,
								void *aux)
	{
		struct thread *a_thread = list_entry (a, struct thread, elem);
		struct thread *b_thread = list_entry (b, struct thread, elem);
		return a_thread->awake_ticks < b_thread->awake_ticks;
	}
	```
	- thread를 sleep list에 추가해준다.
		- 이 때 삽입 정렬을 통해 가장 빠르게 깨어날 수 있는 스레드를 앞에 놓는다.
	- thread의 깨어날 시간을 thread에 함께 저장해준다.
		- 이를 위해 thread 구조체에 awake_ticks를 추가해준다.
	- thread는 block 처리를 해준다.

#### sleep에서 깨우기

- timer_interrupt에서 sleep 깨우기 
	- 매 tick마다 확인해서, 깨울 수 있는 스레드를 sleep_list에서 제거하고 ready_list에 추가한다.
	```c
	static void
	timer_interrupt (struct intr_frame *args UNUSED) {
		ticks++;
		thread_tick ();
		awaken_sleep_thread (ticks);
	}
	```
- 깨우기
	- 잠자고 있는 스레드가 깨어날 시간에 도달했을 때, pop해준다.
	- 이 때 pop 가능한 모든 스레드를 pop해주기 위해 while 문을 사용한다.
	- 시간으로 오름차순 정렬했기 때문에, pop할 수 없는 스레드를 만났을 때 break로 while문을 종료한다.
	```c
	void
	awaken_sleep_thread (int64_t current_ticks) {

		bool is_empty = list_empty(&sleep_list);

		if (is_empty) {
			return;	
		}

		struct thread *first = list_entry (list_front (&sleep_list), struct thread, elem);

		if (first->awake_ticks > current_ticks) {
			return;
		}

		while (!list_empty(&sleep_list))
		{
			first = list_entry (list_front (&sleep_list), struct thread, elem);
			if (first->awake_ticks <= current_ticks)
			{
				list_pop_front (&sleep_list);
				thread_unblock(first);
			} else {
				break;
			}
		}
	}

	```
