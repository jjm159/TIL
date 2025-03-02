# 고 런타임
- Go 프로그램의 실행을 관리하는 핵심 시스템
- 메모리 관리, 고루틴 스케줄링, 네트워크 I/O, 시스템 호출 처리 등을 담당

## 역할
Go 런타임은 운영체제(OS)와 사용자 코드 사이에서 중요한 역할 수행
- 메모리 관리 (GC) → 가비지 컬렉션(자동 메모리 해제)
- 고루틴 스케줄링 → OS 스레드 대신 가벼운 고루틴을 사용
- 네트워크 I/O 관리 → epoll/kqueue/IOCP를 활용한 논블로킹 I/O
- 시스템 호출 Wrapping → OS API 호출을 추상화
    - 운영체제와 독립적인 코드 실행 가능 → Windows, macOS, Linux에서 같은 코드가 동작
- 스택 자동 확장 → 필요하면 스택 크기를 동적으로 증가

## OS가 보는 Go 실행 구조
- Go는 시스템과 애플리케이션 사이에 런타임 계층이 추가된다
```
[Kernel Mode]  ───────────────────────────────────
     │
     │  (system call) 
     ▼
[User Mode] ─────────────────────────────────────
    ├── Go Runtime  ── (epoll/kqueue/IOCP) ──► OS Kernel
    │
    ├── Go Application (main.go 실행)
```
- 개발자가 실행하고자 하는 프로그램과 OS 사이에 추가로 만든 레이어
- Go언어에서 더 시스템을 잘 활용하기 위해 다양한 기능들을 제공하기 위해 만든 레이어
- OS 입장에서는 그저 유저 모드에서 돌아가는 사용자 프로그램일 뿐
- 개발자 입장에서는 내부 구현을 신경 쓰지 않고 그저 시스템 자원을 사용하기 위한 API로서 사용
- 하지만 Go 런타임에 대해 이해를 해야 시스템을 바로 사용하는 것과 비교해서 어떤 차이가 있는지를 알고 개발할 수 있음

## C와 비교
- C의 실행 구조
    - [Application Code] → (직접 `syscall` 호출) → [Operating System Kernel]
    - C는 OS의 syscall을 직접 호출해서 I/O, 프로세스, 메모리 관리를 수행
    - 네트워크를 다룰 때 epoll, select 같은 논블로킹 I/O 처리 코드를 직접 구현해야 함
- Go의 실행 구조
    - [Application Code] → [Go Runtime] → (런타임이 `syscall` 관리) → [Operating System Kernel]
    - Go는 syscall을 직접 호출하지 않고 Go 런타임을 통해 시스템 호출을 사용
    - Go 런타임이 고루틴, I/O 멀티플렉싱, 가비지 컬렉션 등 추가적인 기능을 제공
    - 개발자는 논블로킹 I/O(epoll/kqueue/IOCP)를 직접 다룰 필요 없이 net 패키지만 사용하면 됨
- C에서는 아래와 같이 시스템 콜을 직접 실행
```c
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <unistd.h>

int main() {
    int sockfd = socket(AF_INET, SOCK_STREAM, 0); // 시스템 호출 (syscall)
    struct sockaddr_in server;
    server.sin_family = AF_INET;
    server.sin_addr.s_addr = INADDR_ANY;
    server.sin_port = htons(8080);

    bind(sockfd, (struct sockaddr*)&server, sizeof(server)); // 시스템 호출 (syscall)
    listen(sockfd, 5); // 시스템 호출 (syscall)
    
    int clientfd = accept(sockfd, NULL, NULL); // 시스템 호출 (syscall)
    char buffer[1024];
    read(clientfd, buffer, sizeof(buffer)); // 시스템 호출 (syscall)
    write(clientfd, "Hello", 5); // 시스템 호출 (syscall)

    close(sockfd); // 시스템 호출 (syscall)
    return 0;
}
```
- Go에서는 런타임을 통해 시스템 호출을 간접 사용
```go
package main

import (
    "fmt"
    "net"
)

func main() {
    listener, _ := net.Listen("tcp", ":8080")
    conn, _ := listener.Accept()
    buf := make([]byte, 1024)
    conn.Read(buf) // Go 런타임이 epoll 기반 비동기 처리
    conn.Write([]byte("Hello"))
    conn.Close()
}
```


## 참고
- [go runtime](https://medium.com/@jamal.kaksouri/exploring-the-power-and-flexibility-of-the-go-runtime-9a83f33001cf)