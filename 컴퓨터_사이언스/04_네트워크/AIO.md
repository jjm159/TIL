# AIO - Asyncronous Input Output (비동기 IO)
- 네트워크에 분류하긴 했지만, 꼭 네트워크가 아니어도 파일 인터페이스로 접근 가능한 IO 디바이스 동작들 모두에 적용 가능함
- AIO는 네트워크 인터페이스가 아니라 파일 작업 인터페이스임!

## 커널 스레드를 활용한 비동기 I/O(AIO)
- 최신 프로그래밍 언어들은 운영체제(OS)의 커널 스레드(Kernel Thread)를 활용하여, 
- 비동기 I/O(Asynchronous I/O, AIO)를 처리하는 방식을 사용
- __운영체제가 제공하는 비동기 입출력 시스템 콜__
- I/O가 완료될 때까지 기다리지 않고 다른 작업을 수행하는 방식
- 끝나면 callback을 실행시켜줌

## AIO 지원이 없다면
- 유저 프로그램에서 스레드를 생성하고, IO가 끝나고 나면 콜백함수를 실행시켜주는 방식으로 처리 가능
- 이렇게 해도 비동기 IO 구현이 가능함
- 유저가 할당한 스레드에서 블로킹 된 상태로 IO 작업을 실행하고, 원래 스레드는 논블락킹으로 원래 실행흐름 그대로 실행
- 이렇게 하면 원래 메인 스레드의 응답성은 좋아져도, 
- __IO를 처리하는 스레드는 블락되어 CPU가 busy wait를 하게 됨__

## 비동기 I/O(AIO) 방식
- 1000개의 I/O 요청이 있어도, OS의 이벤트 시스템이 처리함
- CPU는 다른 유용한 작업을 수행 가능
- 메모리 사용량이 적고, 대량의 I/O 요청을 처리하기에 적합

## 구체적으로 어떻게 동작하는겨?
- __CPU가 IO 작업을 하지 않는게 핵심임__
- 장치 드라이버가 IO 작업을 수행
- IO 작업이 끝나면 __인터럽트__ 발생
- CPU가 인터럽트 핸들러에서, 해당 장치드라이버의 작업이 끝났을 때 실행할 콜백을 실행해줌

## 이게 가능하려면 필요한 기술
- DMA (Direct Memory Access) 
    - CPU 개입 없이 메모리 직접 접근
    - CPU가 데이터를 직접 복사하지 않고, DMA 컨트롤러가 대신 복사하여 CPU 오버헤드를 줄임
- 인터럽트 (Interrupt) 
    - CPU는 I/O가 끝날 때까지 **바쁜 대기(Busy Waiting)**를 하지 않음
    - 대신, I/O가 완료되면 장치가 인터럽트를 발생시켜 CPU에게 알림

## POSIX 예시
```c
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <aio.h>
#include <unistd.h>
#include <string.h>

// AIO 완료 후 실행될 콜백 함수
void aio_completion_handler(sigval_t sigval) {
    struct aiocb *req = (struct aiocb *)sigval.sival_ptr;
    if (aio_error(req) == 0) {
        printf("AIO Read Completed: %s\n", (char *)req->aio_buf);
    }
}

int main() {
    int fd = open("test.txt", O_RDONLY);
    if (fd < 0) {
        perror("open");
        return 1;
    }

    struct aiocb aio;
    memset(&aio, 0, sizeof(struct aiocb));

    aio.aio_fildes = fd;
    aio.aio_buf = malloc(100);
    aio.aio_nbytes = 100;
    aio.aio_offset = 0;
    aio.aio_sigevent.sigev_notify = SIGEV_THREAD; // AIO 완료 후 쓰레드에서 실행
    aio.aio_sigevent.sigev_notify_function = aio_completion_handler;
    aio.aio_sigevent.sigev_value.sival_ptr = &aio;

    // aio_read는 IO를 기다리지 않고, 바로 반환됨
    if (aio_read(&aio) == -1) {
        perror("aio_read");
        return 1;
    }

    // 아직 IO가 끝나지 않았는데 main이 끝나면 안되니까 sleep으로 기다림
    printf("AIO 요청 보냄, 다른 작업 수행 중...\n");
    sleep(3); // AIO가 완료될 때까지 다른 작업 수행

    free((void *)aio.aio_buf);
    close(fd);
    return 0;
}
```

## 효과
- 스레드 생성 안해도 됨
- 컨텍스트 스위치 비용 없음
- CPU를 효율적으로 사용(비지 웨이트 문제 해결)
- 대규모 네트워크 프로그래밍에서 굳

## 리눅스 비동기 I/O API
- POSIX AIO (aio_read(), aio_write())
    - 기존의 POSIX 표준 AIO 방식이지만 성능이 떨어짐
    - aio_read(), aio_write() 호출 후 aio_error(), aio_return()으로 상태 확인
- io_uring (최신 AIO)
    - 리눅스 5.1 이상에서 도입된 최신 AIO 시스템
    - 커널과 유저 공간 간의 ring buffer를 사용하여 성능 최적화
    - 기존 AIO보다 훨씬 빠르고 효율적
    ```c
    #include <liburing.h>
    #include <fcntl.h>
    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
    #include <unistd.h>

    int main() {
        struct io_uring ring;
        io_uring_queue_init(8, &ring, 0);

        int fd = open("test.txt", O_RDONLY);
        struct io_uring_sqe *sqe = io_uring_get_sqe(&ring);
        struct io_uring_cqe *cqe;
        
        char buf[100] = {0};
        struct iovec iov = { .iov_base = buf, .iov_len = sizeof(buf) };
        
        io_uring_prep_readv(sqe, fd, &iov, 1, 0);
        io_uring_submit(&ring);
        io_uring_wait_cqe(&ring, &cqe);
        
        printf("Read: %s\n", buf);
        
        io_uring_cqe_seen(&ring, cqe);
        io_uring_queue_exit(&ring);
        close(fd);
    }
    ```

## 리눅스 이벤트 드리븐 IO
- epoll
    - 파일 디스크립터(FD)들의 I/O 상태 변화를 감지하는 시스템
    - 또 다른 이벤트 드리븐 IO 방법인 select()보다 성능이 훨씬 좋음
        - 커널이 모든 FD에 대해 검사하면서 대기 -> 성능 저하
- 이벤트가 발생한 FD만 감지하여 빠르고 확장성이 좋음
```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/epoll.h>
#include <fcntl.h>
#include <unistd.h>

#define MAX_EVENTS 10

int main() {
    int epfd = epoll_create1(0);
    struct epoll_event event, events[MAX_EVENTS];
    
    event.events = EPOLLIN;
    event.data.fd = STDIN_FILENO;
    epoll_ctl(epfd, EPOLL_CTL_ADD, STDIN_FILENO, &event);

    while (1) {

        // 여기서 블락됨, 이벤트가 감지되면 그 때 블락이 풀리면서 아래 코드 실행
        // 여기서 블락되기 때문에 busy wait가 아님
        int num_events = epoll_wait(epfd, events, MAX_EVENTS, -1);

        for (int i = 0; i < num_events; i++) {
            char buffer[100];
            read(events[i].data.fd, buffer, sizeof(buffer));
            printf("Received: %s\n", buffer);
        }
    }

    close(epfd);
    return 0;
}
```
- epoll_wait()는 데이터가 준비되면 이벤트를 발생시켜 CPU 리소스를 절약
- 이벤트 감지만 수행하며, 실제 I/O는 read()/write()로 직접 수행해야 함
- epoll_wait()
    - 여기서 블락이 됨
    - thread가 블락 리스트에 들어가서 busy wait가 아님
    - 이벤트가 발생하면 그 때 블락이 풀림
- io 작업이 많을 수록 효과 굳 - O(1)에 처리
    - 모든 fd를 검사하면서 대기하지 않음

- go언어 웹소켓에서는 어떻게 하고 있나?


## 윈도우는 좀 신기함 - IOCP
- Windows의 IOCP (I/O Completion Ports, AIO + 이벤트 드리븐 I/O)
- AIO와 이벤트 감지 기능을 모두 포함한 독특한 방식
    - AIO 기능: 커널이 직접 비동기 I/O를 수행하고 완료되면 알림
    - 이벤트 드리븐 기능: I/O가 준비되었을 때 이벤트를 감지하여 알려줌
- 실제 게임 서버에서 IOCP를 사용하여 여러 소켓 연결을 처리함!!!
- code
```c
#include <windows.h>
#include <stdio.h>

void CALLBACK FileIOCompletionRoutine(DWORD errorCode, DWORD bytesTransferred, LPOVERLAPPED lpOverlapped) {
    printf("Asynchronous read completed: %d bytes read\n", bytesTransferred);
}

int main() {
    HANDLE file = CreateFile("test.txt", GENERIC_READ, 0, NULL, OPEN_EXISTING, FILE_FLAG_OVERLAPPED, NULL);
    char buffer[100];
    OVERLAPPED overlapped = {0};

    ReadFileEx(file, buffer, sizeof(buffer), &overlapped, FileIOCompletionRoutine);

    SleepEx(INFINITE, TRUE); // 비동기 I/O 완료될 때까지 대기
    CloseHandle(file);
    return 0;
}
```
- IOCP는 파일, 네트워크, 소켓 I/O에도 사용할 수 있어 매우 강력함
- Linux의 io_uring과 비슷한 개념이지만, Windows 전용임

## AIO vs event driven IO
- AIO는 커널이 직접 I/O를 수행하고 완료되면 알림
    - DMA와 인터럽트를 사용하여!!
    - callback을 넘겨줌
- 이벤트 드리븐 I/O는 I/O 가능 여부만 감지하고, 응용 프로그램이 직접 I/O를 수행
    - IO 장치 사용이 가능해지면 그 때 응용 프로그램이 직접 IO를 수행
- 중요한건 CPU가 계속 확인하지 않는다는 것이야!!!
    - 외부 장치가 작업하는 동안 CPU는 계속 감시하지 않음!!!
    - 외부 장치가 인터럽트를 통해 알려주기 때문에 이게 가능함!!! - 외부 장치의 도움!!!


## 참고
- [AIO](https://djkeh.github.io/articles/Boost-application-performance-using-asynchronous-IO-kor/)
- [IOCP](https://thebook.io/006884/0219/)