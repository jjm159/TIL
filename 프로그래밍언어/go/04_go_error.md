# 에러 처리

- [go에는 try-catch가 없다](https://blog.naver.com/sjc02183/222927597790)

## try-catch가 없다
- 왜?
- try-catch는 에러를 미루게 된다. 책임을 전가해버린다.
- 에러 처리가 많아지고 복잡한 세상에서 누군가 해주겠지 하고 던지는 것을 허용하는 문법
- go는 이게 싫은 사람이 만든 언어
- error가 났다면, 거기서 책임을 지고 처리를 해야 한다는 것

## panic, recover
- 책임을 돌리지 않는다.
- panic이 난 위치에서 해결하도록 한다.
- 개발자에게 에러 처리에 대한 책임을 부여해 해당 에러를 어디서, 누가, 어떻게 처리해야 할 지 더 심도있게 고민하도록 유도
- 사용자에게 자율성을 보장하면서도, 생각할 여지를 남겨주는 셈
- panic 이라는 단어의 선정도 매우 탁월하다고 할 수 있는데, 
    - try-recover 와 달리 panic-recover 는 단어 만으로
    - 예외 상황이 아닌 명백한 에러 상황에서만 사용해야만 한다는 압박을 개발자에게 부여
    - 자연스레 개발자는 recover 를 남용하지 않고 필요한 곳에서만 사용하게 된다. 이는 go 에서 간결한 코드를 짜는 데에 큰 도움

# recover 사용하기
```go
package main

import "fmt"

// 패닉이 일어날 가능성이 있는 함수
func mayPanic(shouldPanic bool) {
	if shouldPanic {
		panic("Something went wrong!")
	}
	fmt.Println("No panic, everything is okay.")
}

// recover를 통해 panic을 복구하는 함수
func handlePanic(shouldPanic bool) {
	defer func() {
		// panic이 발생하면 이 defer 함수에서 recover를 호출
		if r := recover(); r != nil {
			fmt.Println("Recovered from panic:", r)
		}
	}()

	mayPanic(shouldPanic)
	fmt.Println("Function handlePanic completed without panic.")
}

func main() {
	fmt.Println("=== Case 1: No Panic ===")
	handlePanic(false)

	fmt.Println("\n=== Case 2: Panic Occurs ===")
	handlePanic(true)
	fmt.Println("Program continues after recovering from panic.")
}
```

