# Go... 렛츠 Go...

## 설치 실행
- 공식홈페이지 설치 
- 터미널 버전 확인
    - go version
- Go 작업 폴더 설정
    - mkdir hello-go
    - cd hello-go
- Go 모듈 초기화
    - go mod init hello-go
- Go 모듈 가져오기
    - go mod tidy
- Go 실행
    - go run .
- Go 빌드
    - go build
- Go test
    - go test

---

## 다른 파일에 있는 코드 사용하기 
- import 해주기
    - go module을 import 하는 경우에는 go.main에 경로 추가해줘야 함
- alias할 수 있음
    - . 으로 해서 모두 가져올 수도 있는데, 이건 권장하는 방식은 아님
```go
package main

import (
	"fmt"
	r "go_routine/modules/rectangle"
	"go_routine/utils"
)

func main() {
	rectangle := r.NewRectangle(1, 3)
	result := utils.Add(3, 5)
	fmt.Println("Hello", result, rectangle.Area())
}

```

## 구조체 사용 하기
- 대문자는 public, 소문자는 private
- pointer receiver와 value receiver 차이
```go
package modules

type Rectangle struct {
	width  int
	height int
}

// 생성자
func NewRectangle(width int, height int) *Rectangle {
	return &Rectangle{width: width, height: height}
}

// method
func (r Rectangle) Area() int {
	return r.width * r.height
}


func (r *Rectangle) SetWidth(width int) {
	r.width = width
}

// 이렇게 하면 현재 스택에 있는 r의 height만 변경
func (r Rectangle) SetHeight(height int) {
	r.height = height
}

```

## 비동기
- [고루틴](http://golang.site/go/article/21-Go-%EB%A3%A8%ED%8B%B4-goroutine)
    - 경량 스레드
    - Go는 디폴트로 1개의 CPU 사용. 그래서 goroutine도 하나의 cpu를 나눠서 쓰는 concurrent가 디폴트.
    - 동시 처리가 아니라 병렬 처리를 위해선 CPU 수 조절 필요
        ```Go
        runtime.GOMAXPROCS(4)
        ```
- [waitgroup](https://gobyexample.com/waitgroups)
    - 모든 고루틴의 종료를 기다림
    ```Go
    var wg sync.waitgroup
    wg.Add(1)
    go func() {
        defer wg.Done()
        time.Sleep(time.Second)
    }
    wg.Add(2)
    go func() {
        defer wg.Done()
        time.Sleep(time.Second)
    }
    wg.Wait() // wg가 Done을 모두 호출하여 0이 될 때까지 Block
    ```

## 채널 사용하기
- 고루틴간 데이터 전송 방법
- 큐! 앞에서부터 순서대로! 하나씩! 동시 X
- [채널](https://etloveguitar.tistory.com/40)

## protobuf
- 설치
    - brew install protobuf
- 환경 설정
    - go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
    - export PATH=$PATH:$(go env GOPATH)/bin
- 변환
    - protoc --go_out=. --go_opt=paths=source_relative ./message/message.proto
- 유니티에서 사용하기
    - 라이브러리 설치
        - dotnet add package Google.Protobuf
    - Asset에 protobuf 파일 추가
    - script에 protobuf cs 파일 생성
        - protoc --proto_path=Assets/Proto --csharp_out=Assets/Scripts/Proto Assets/Proto/message.proto

---

# 참고 자료
- https://medium.com/eureka-engineering/understanding-allocations-in-go-stack-heap-memory-9a2631b5035d
- https://mclub4.tistory.com/47

