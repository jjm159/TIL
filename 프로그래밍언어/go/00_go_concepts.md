# Go언어 컨셉

- 언어를 배울 때 언어가 어떤 목적으로 만들어졌는지 알면 빠르게 배울 수 있다.
- 그 목적에 맞는 concept가 있을 것이고, 이런 컨셉들을 이해하면 각 문법이 자연스럽게 이해가 된다.
- concept와 관련해서 좋은 글이 있어 해당 글을 요약해보았다.
- [참고](https://medium.com/@zakariasaif/guide-to-programming-paradigms-in-golang-go-eff42b678a40)

## Go언어 개요

- Go __단순성__, __효율성__, __사용 편의성__ 을 위해 설계된 정적 타입 언어
- 확장 가능하고 동시적이며 효율적인 시스템을 만드는데 뛰어남
- 웹서버, 네트워크 도구, 대규모 분산 시스템 구축에 적합
- 단순성, 고성능, 강력한 동시성 지원의 조합 Good

## Go는 기본적으로 명령형/절차형 프로그래밍
- 프로그램의 목적 달성에 필요한 각 프로세스를 개발자가 명시적으로 지정
```go
package hello

import "fmt"

func main() {
    // 단계를 명시적으로 정의
    fmt.Println("hello")
    fmt.Println("world")
}
```
- 절차적 프로그래밍은 명령형 프로그래밍의 하위 집합
- 프로그램이 프로시저로 구성되어 Go에서는 함수라고 함
- 명령형/절차형 프로그래밍의 주요 특징
    - if 
        - 명확한 제어 흐름
    - 모듈성 
        - 함수로 작고 관리하기 쉬운 부분으로 나눔
    - 변수 조작 
        - 데이터를 저장하고 수정하기 위해 변수를 직접 조작
    - 구조적 프로그래밍 
        - block, loop, procedure를 사용하여 명확하고 체계적인 코드 강조

## Go의 동시 프로그래밍
- 동시 프로그래밍은 여러 작업을 동시에 실행하여 소프트웨어 시스템에서 리소스를 보다 효율적으로 사용하고 응답성을 개선하는 패러다임
- 문제를 작고 독립적인 작업으로 나눠 동시에 실행할 수 있도록 함
- Go는 동시 프로그래밍을 위해 기본으로 다양한 기능을 제공함
- 고루틴
    - 고루틴은 가볍고 독립적으로 예약된 실행 스레드
    - go 키워드를 사용하여 동시 작읍을 쉽게 시작
- 채널
    - 채널을 사용하여 고루틴 끼리 서로 통신
    - 동시 작업 사이에서 데이터를 전달하는 방법
        - 지저분한 락처리를 하지 않고 쉽게 데이터 공유
- 동시성 패턴
    - 여러 채널을 처리하기 위해 팬아웃, 팬인, 워커 풀, select 문 등 다양한 동시성 패턴 지원

## 함수형 프로그래밍
- 계산을 수학적 함수의 평가로 취급하고 변화하는 상태와 가변 데이터를 피하는 프로그래밍 패러다임
- Go는 순수 함수형 프로그래밍 언어는 아니지만 일부 함수형 프로그래밍 개념과 기능을 통합함
- 일급함수 First-Class Functions
    - Go는 일급 함수를 지원함
    - 함수를 변수에 할당하고, 다른 함수에 인수로 전달하고, 다른 함수에서 값으로 반환할 수 있음
- 고차함수
    - 다른 함수를 인수로 받거나 반환하는 함수를 고차함수라고 하는데 Go도 이를 허용함
- 불변성
    - 변수의 변형을 허용하긴 하지만, 함수형 프로그래밍은 불변성을 장려함
    - 상수를 사용하고 직접 변형을 피함으로써 불변성을 달성할 수 있음
- 익명 함수(클로저)
    - 클로저라고 하는 익명 함수를 지원
    - 주변 범위에서 변수를 캡처하고 참조할 수 있음
- 불변 데이터 구조
    - 기본적으로 변경 불가능한 데이터 구조를 제공하지는 않지만,
    - 슬라이스나 다른 복합 유형을 사용하여 구현할 수 있음

## 객체 지향 프로그래밍(OOP)
- 소프트웨어 설계를 객체의 개념을 중심으로 구성하는 프로그래밍 패러다임
- 객체는 실제 세계의 엔터티 또는 개념
- 객체는 클래스의 인스턴스이며, 클래스는 해당 유형의 객체가 나타낼 수 있는 속성(속성)과 동작(메서드)을 정의
- Go는 **순수한 객체 지향 언어는 아니고** 클래스 __상속과 같은 일부 전통적인 OOP 기능이 없지만__, 
- 더 가볍고 유연한 방식으로 객체 지향 원칙을 지원

#### 1. 객체로서의 구조체:
- Go는 구조체를 사용하여 유형을 정의하고, 메서드는 이러한 유형과 연관
- 클래스는 아니지만 구조체는 데이터와 연관된 동작을 캡슐화하는 방법을 제공
```go
패키지 main 

import  "fmt" 

// 객체 
유형을 나타내는 구조체 Circle struct { 
    Radius float64
 } 

// 구조체와 연관된 메서드 
func  (c Circle) Area() float64 { 
    return  3.14 * c.Radius * c.Radius 
} 

func  main () { 
    // Circle 유형의 객체 생성
     myCircle := Circle{Radius: 5 } 

    // 객체에서 메서드 호출
     fmt.Println( "Circle Area:" , myCircle.Area()) 
}
```
#### 2. 다형성을 위한 인터페이스
- Go는 메서드 집합을 정의하는 인터페이스를 지원
- 유형은 인터페이스를 암묵적으로 충족하여 인터페이스 구현을 명시적으로 선언하지 않고도 다형성 동작을 허용
```go
패키지 main 

import  "fmt" 

// 공통적인 동작을 정의하는 인터페이스 
유형 Shape 인터페이스 { 
    Area() float64
 } 

// 인터페이스를 구현하는 구조체 
유형 Circle struct { 
    Radius float64
 } 

func  (c Circle) Area() float64 { 
    return  3.14 * c.Radius * c.Radius 
} 

유형 Square struct { 
    SideLength float64
 } 

func  (s Square) Area() float64 { 
    return s.SideLength * s.SideLength 
} 

func  printArea (shape Shape) { 
    fmt.Println( "Shape Area:" , shape.Area()) 
} 

func  main () { 
    // 다른 유형의 객체 생성
     myCircle := Circle{Radius: 5 } 
    mySquare := Square{SideLength: 4 } 

    // 인터페이스와 함께 다형성 사용
     printArea(myCircle) 
    printArea(mySquare) 
}
```
#### 3. 코드 재사용을 위한 구성:
- Go는 상속보다 합성을 장려
- 개발자는 클래스 계층에 의존하는 대신 구조체를 사용하여 type을 구성
- 코드 재사용을 위해 한 구조체를 다른 구조체에 내장할 수 있음
    - 합성!!!
    - has-a
```go
패키지 main 

import  "fmt" 

// 부모 구조체 
유형 Shape struct { 
    Color string
 } 

// 부모 구조체를 내장하는 자식 구조체 
유형 Circle struct { 
    Shape   // Shape 구조체 내장
     Radius float64
 } 

func  main () { 
    // 구성을 가진 객체 생성
     myCircle := Circle{Shape: Shape{Color: "Red" }, Radius: 5 } 

    // 내장된 구조체에서 속성 액세스
     fmt.Println( "Circle Color:" , myCircle.Color) 
}
```

## 선언적 프로그래밍
- 제어 흐름이나 컴퓨터가 프로그램을 실행하기 위해 취해야 하는 단계를 명시적으로 설명하지 않고 
- 계산의 논리를 표현하는 프로그래밍 패러다임
- 특정 결과를 달성하는 방법에 초점을 맞추는 대신 원하는 결과가 무엇인지 강조
- 솔루션을 달성하기 위한 단계별 절차를 설명하는 대신 솔루션이 충족해야 하는 속성이나 제약 조건을 지정하는 것을 포함
- Go는 개발자가 더 선언적인 스타일로 코드를 작성할 수 있도록 하는 기능과 패턴을 제공

#### 1. 구조체를 사용한 선언적 코드
- Go의 구조체 사용은 선언적 방식으로 복잡한 데이터 구조를 정의할 수 있게 해줌
- 조작 방법을 지정하지 않고도 데이터의 구조와 속성을 정의
```go
// 선언적 구조체 정의 
type Person struct { 
    FirstName string
     LastName   string
     Age        int
 } 

// 인스턴스 생성
 p := Person{ 
    FirstName: "John" , 
    LastName:   "Doe" , 
    Age:        30 , 
}
```

#### 2. Gorilla Mux를 사용한 선언적 HTTP 라우팅
- Go에서 웹 애플리케이션을 사용할 때 Gorilla Mux와 같은 프레임워크를 사용하면 선언적 방식으로 경로와 핸들러를 선언할 수 있음
```go
r := mux.NewRouter() 

// 선언적 경로 정의
 r.HandleFunc( "/users/{id:[0-9]+}" , GetUserHandler).Methods( "GET" )
```

#### 3. html/template을 사용한 선언적 템플릿 렌더링
- Go의 패키지 html/template를 이용하면 HTML 템플릿의 선언적 정의가 가능해져 구조와 데이터를 분리할 수 있음
```go
// 선언적 HTML 템플릿
 tmpl := ` 
<html> 
<head><title>{{.Title}}</title></head> 
<body> 
    <h1>{{.Header}}</h1> 
    <ul> 
    {{range .Items}} 
        <li>{{.}}</li> 
    {{end}} 
    </ul> 
</body> 
</html>
```
#### 4. SQL을 사용한 선언적 데이터베이스 쿼리
- Go database/sql패키지를 사용하면 매개변수화된 쿼리를 사용하여 데이터베이스 쿼리에 대한 보다 선언적인 접근 방식을 사용할 수 있음
```go
// 선언적 SQL 쿼리
 행, err := db.Query( "SELECT name, age FROM users WHERE age > ?" , 21 )
```

- Go는 순수한 선언적 언어로 간주되지 않을 수 있지만, 적절한 경우 기능과 사용 가능한 라이브러리를 활용하여 더 선언적 스타일로 코드를 작성할 수 있음

## 정리
- Go는 다양한 패러다임 지원
- 특정 프로그램이나 프로젝트의 요구 사항에 따라 개별적으로 또는 조합하여 사용
- Go의 단순성과 가독성에 대한 집중은 다양한 프로그래밍 스타일에 적합하고 다재다능하게 함