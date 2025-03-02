# go websocket
- 내부적으로 어떻게 네트워크 IO를 처리하고 있는지?
- 성능을 극대화하기 위해 어떻게 코딩해야 하는지?

## 내부 코드 살펴보기
- 내부적으로 이벤트 드리븐 방식의 네트워크 IO를 사용하고 있다.
- 소켓 디바이스를 무작정 기다리는 게 아니라, 사용 가능할 때 이벤트가 발생하고 이를 받아 처리하는 것
- 그래서 CPU를 최적화해서 사용할 수 있다!
- 내부 코드 들어가보자!
- tcp connection
```go
var upgrader = websocket.Upgrader{
    ReadBufferSize:  8192, // 읽기 버퍼 크기
    WriteBufferSize: 8192, // 쓰기 버퍼 크기
    CheckOrigin: func(r *http.Request) bool {
        return true
    },
}
```
- 여기 Websocket 라이브러리에서 제공해주는 Upgrader를 따라가보면, Conn이라는 구조체가 선언되어 있음
```go
type Conn struct {
	conn        net.Conn
	isServer    bool
	subprotocol string
    ...
```
- net.Conn을 따라 들어가면
```go
// Conn is a generic stream-oriented network connection.
//
// Multiple goroutines may invoke methods on a Conn simultaneously.
type Conn interface {
	// Read reads data from the connection.
	// Read can be made to time out and return an error after a fixed
	// time limit; see SetDeadline and SetReadDeadline.
	Read(b []byte) (n int, err error)

	// Write writes data to the connection.
	// Write can be made to time out and return an error after a fixed
	// time limit; see SetDeadline and SetWriteDeadline.
	Write(b []byte) (n int, err error)
    ...
```
- 이렇게 Read, Write 인터페이스가 있음
- 이 파일 안에서 Write를 검색해보면 구현체를 찾을 수 있음
```go
// Write implements the Conn Write method.
func (c *conn) Write(b []byte) (int, error) {
	if !c.ok() {
		return 0, syscall.EINVAL
	}
	n, err := c.fd.Write(b)
	if err != nil {
		err = &OpError{Op: "write", Net: c.fd.net, Source: c.fd.laddr, Addr: c.fd.raddr, Err: err}
	}
	return n, err
}
```
- 여기서 다시 c.fd.Write를 따라 들어가면
```go
func (fd *netFD) Write(p []byte) (nn int, err error) {
	nn, err = fd.pfd.Write(p)
	runtime.KeepAlive(fd)
	return nn, wrapSyscallError(writeSyscallName, err)
}
```
- fd.pfd.Write(p)를 다시 따라 들어가면
```go

// Write implements io.Writer.
func (fd *FD) Write(p []byte) (int, error) {
	if err := fd.writeLock(); err != nil {
		return 0, err
	}
	defer fd.writeUnlock()
	if err := fd.pd.prepareWrite(fd.isFile); err != nil {
		return 0, err
	}
	var nn int
	for {
		max := len(p)
		if fd.IsStream && max-nn > maxRW {
			max = nn + maxRW
		}
		n, err := ignoringEINTRIO(syscall.Write, fd.Sysfd, p[nn:max])
		if n > 0 {
			if n > max-nn {
				// This can reportedly happen when using
				// some VPN software. Issue #61060.
				// If we don't check this we will panic
				// with slice bounds out of range.
				// Use a more informative panic.
				panic("invalid return from write: got " + itoa.Itoa(n) + " from a write of " + itoa.Itoa(max-nn))
			}
			nn += n
		}
		if nn == len(p) {
			return nn, err
		}
		if err == syscall.EAGAIN && fd.pd.pollable() {
			if err = fd.pd.waitWrite(fd.isFile); err == nil {
				continue
			}
		}
		if err != nil {
			return nn, err
		}
		if n == 0 {
			return nn, io.ErrUnexpectedEOF
		}
	}
}
```
- 이게 나옴!!!! 여기 보면 fd.pd.waitWrite 가 나오는데, 이게 이벤트 드리븐 방식이라는 것!!!!
```go
package poll

...

func (pd *pollDesc) waitWrite(isFile bool) error {
	return pd.wait('w', isFile)
}

...

func (pd *pollDesc) wait(mode int, isFile bool) error {
	if pd.runtimeCtx == 0 {
		return errors.New("waiting for unsupported file type")
	}
	res := runtime_pollWait(pd.runtimeCtx, mode)
	return convertErr(res, isFile)
}
```
- runtime_pollWait는 Go의 런타임 패키지 함수
- 실제 구현체에서는 OS에 따라 다른 방식으로 poll을 구현하고 있음
    - 리눅스에서는 epoll, macOS에서는 kqueue, 윈도우에서는 IOCP를 사용
    - 외부 장치의 도움(인터럽트)으로 Non-Block I/O를 사용하여 CPU 낭비를 줄인다!!!
- 요약
    - gorilla/websocket은 내부적으로 netFD를 사용해서 소켓을 관리
    - netFD는 internal/poll.FD를 활용하여 비동기 네트워크 이벤트 감시
    - 리눅스에서는 epoll, macOS에서는 kqueue, 윈도우에서는 IOCP를 사용
    - poll_runtime_pollWait이 내부적으로 epoll_wait을 호출하여 소켓 상태를 감시
    - 이벤트 발생 시, 고루틴이 깨어나서(unblock) 실행 → 이벤트 드리븐 방식
- 그래서 빠르다!!!
    - 내부적으로 이벤트 드리븐 IO 방식으로 동작하기 때문에 쓸데없이 CPU 자원을 낭비하지 않는다!

## 기본 동기적 처리
```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"github.com/gorilla/websocket"
)

var upgrader = websocket.Upgrader{
	CheckOrigin: func(r *http.Request) bool { return true },
}

func handleWebSocket(w http.ResponseWriter, r *http.Request) {
	conn, err := upgrader.Upgrade(w, r, nil)
	if err != nil {
		log.Println("WebSocket Upgrade Error:", err)
		return
	}
	defer conn.Close()

	// 동기적으로 메시지 읽기 (Blocking)
	for {
		_, msg, err := conn.ReadMessage()
		if err != nil {
			log.Println("Read Error:", err)
			break
		}
		fmt.Println("Received:", string(msg))

		// 동기적으로 메시지 응답 (Blocking)
		err = conn.WriteMessage(websocket.TextMessage, []byte("Message received"))
		if err != nil {
			log.Println("Write Error:", err)
			break
		}
	}
}

func main() {
	http.HandleFunc("/ws", handleWebSocket)
	log.Println("WebSocket Server running on :8080")
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```
- Websocket 내부적으로 소켓 IO를 이벤트 드리븐으로 처리하긴 하지만, 
- Websocket에서 제공하는 WriteMessage와 같은 API는 블락킹 동기 방식으로 동작함
    - block된다는 건, 현재 스레드(고루틴)가 스케줄링의 대상이 아닌 상태가 된다는 말
- 그래서 여러개의 연결을 동시에 처리하려면 각 연결마다 고루틴을 따로 생성해주어야 함

## 고루틴, 채널 사용
- 고루틴(Goroutine)과 채널(Channel)을 사용하여 비동기적으로 메시지를 처리
```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"github.com/gorilla/websocket"
)

var upgrader = websocket.Upgrader{
	CheckOrigin: func(r *http.Request) bool { return true },
}

func handleWebSocket(w http.ResponseWriter, r *http.Request) {
	conn, err := upgrader.Upgrade(w, r, nil)
	if err != nil {
		log.Println("WebSocket Upgrade Error:", err)
		return
	}

	// 메시지 송수신을 위한 채널 생성
	messageChan := make(chan []byte)

	// 수신 Goroutine
	go func() {
		defer close(messageChan)
		for {
			_, msg, err := conn.ReadMessage()
			if err != nil {
				log.Println("Read Error:", err)
				break
			}
			fmt.Println("Received:", string(msg))
			messageChan <- msg // 채널에 메시지 전달
		}
	}()

	// 송신 Goroutine
	go func() {
		for msg := range messageChan {
			err := conn.WriteMessage(websocket.TextMessage, []byte("Echo: "+string(msg)))
			if err != nil {
				log.Println("Write Error:", err)
				break
			}
		}
	}()

	// 연결이 종료될 때까지 대기
	select {}
}

func main() {
	http.HandleFunc("/ws", handleWebSocket)
	log.Println("WebSocket Server running on :8080")
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```
- 비동기 방식으로 메시지를 처리할 수 있도록 “수신 Goroutine + 송신 Goroutine + Channel”을 활용.
- WebSocket이 동시 요청을 비동기적으로 처리


## netpoll 사용해서 고루틴 줄이기
```go
package main

import (
	"fmt"
	"log"
	"net"
	"net/http"
	"sync"
	"time"

	"github.com/cloudwego/netpoll"
	"github.com/gorilla/websocket"
)

var upgrader = websocket.Upgrader{
	CheckOrigin: func(r *http.Request) bool { return true },
}

// WebSocket 클라이언트 구조체
type WebSocketClient struct {
	conn    *websocket.Conn
	pollFD  netpoll.Poller
	sendBuf chan []byte
}

// WebSocket 메시지 읽기
func (c *WebSocketClient) readMessages() {
	defer c.conn.Close()
	for {
		_, msg, err := c.conn.ReadMessage()
		if err != nil {
			log.Println("Read error:", err)
			break
		}
		fmt.Println("Received:", string(msg))

		// 메시지를 비동기적으로 송신 버퍼에 추가
		c.sendBuf <- msg
	}
}

// WebSocket 메시지 쓰기
func (c *WebSocketClient) writeMessages() {
	for msg := range c.sendBuf {
		err := c.conn.WriteMessage(websocket.TextMessage, msg)
		if err != nil {
			log.Println("Write error:", err)
			break
		}
	}
}

// `netpoll`을 활용한 WebSocket 이벤트 핸들러
func handleWebSocket(conn *websocket.Conn, poller netpoll.Poller) {
	client := &WebSocketClient{
		conn:    conn,
		pollFD:  poller,
		sendBuf: make(chan []byte, 256),
	}

	var wg sync.WaitGroup
	wg.Add(2)

	// 메시지 읽기 및 쓰기를 별도의 Goroutine에서 수행
	go func() {
		defer wg.Done()
		client.readMessages()
	}()

	go func() {
		defer wg.Done()
		client.writeMessages()
	}()

	wg.Wait()
	close(client.sendBuf)
}

// HTTP 핸들러 (WebSocket 업그레이드)
func wsHandler(w http.ResponseWriter, r *http.Request) {
	conn, err := upgrader.Upgrade(w, r, nil)
	if err != nil {
		log.Println("WebSocket upgrade error:", err)
		return
	}

	// `netpoll` 생성
	poller, err := netpoll.CreatePoller()
	if err != nil {
		log.Println("Netpoll error:", err)
		conn.Close()
		return
	}

	// WebSocket 연결을 `netpoll`로 관리
	go handleWebSocket(conn, poller)
}

// 메인 함수
func main() {
	http.HandleFunc("/ws", wsHandler)
	server := &http.Server{
		Addr:         ":8080",
		ReadTimeout:  10 * time.Second,
		WriteTimeout: 10 * time.Second,
	}

	log.Println("WebSocket server running on :8080")
	log.Fatal(server.ListenAndServe())
}
```
- Goroutine을 최소화하고, netpoll을 활용하여 WebSocket을 비동기적으로 처리
- netpoll을 활용하여 epoll/kqueue 기반으로 논블로킹 WebSocket 서버를 구축
- Goroutine을 최소화하여 성능을 최적화
- 대량의 WebSocket 연결을 효과적으로 관리할 수 있음



## 참고
- [go websocket 효율적으로!](https://medium.com/free-code-camp/million-websockets-and-go-cc58418460bb)