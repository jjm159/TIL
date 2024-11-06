# SRP (single responsibility principal)

하나의 모듈은 하나의, 오직 하나의 액터에 대해서만 책임져야 한다. 

여기서 액터란 변경을 요청하는 한 명 이상의 사람을 의미한다. 

변경의 원인이 되는 액터 당 하나의 모듈만 할당되어야 한다. 

```swift
class Employee {
	func calculatePay() { regularHours() } // CFO
	func reportHours() {regularHours() } // COO
	func save() {...} // CTO

	func regularHours() {} // calculatePay와 reportHours의 공통 알고리즘을 함수로.
```

여기서 문제. CFO에서 업무 시간 계산 방식을 수정하여 calculatePay에서 호출한 regularHours를 수정하게 됨. 하지만 COO에서는 계산 방식을 바꾸지 않음. 

→ 서로 다른 액터가 의존하는 코드를 분리하여야 한다. 

→ 모두가 각기 다른 클래스로 이동, 액터가 다른 모듈끼리는 서로 존재를 몰라야 한다. 그래야 우연한 중복을 피할 수 있다. 

```swift
class Employee { ... }
class PayCalculator {
		func calculatePay() { regularHours() } // CFO
}
class HourReporter {
		func reportHours() { regularHours() } // COO
}
class EmployeeSaver {
		func save() {...} // CTO
} 
```


---

### 참고
[클린 아키텍처](https://book.naver.com/bookdb/book_detail.nhn?bid=15303798)

[Clean code - SOLID 01 - Single Responsibility Principle (SRP)](https://tothefullest08.github.io/php/2020/01/29/Cleancode13-srp/)