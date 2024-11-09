# 얕은 복사와 깊은 복사

메모리를 알아서 정리해주는 스위프트나 파이썬 같은 언어도 결국 이 얕은 복사, 깊은 복사 문제로 인해 메모리 고민을 안할 수는 없다. 

연결리스트와 해시 테이블을 구현하면서, 어떻게 보면 c++보다 복잡하다고 생각이 들 정도였다. swift에서 이 문제는 다음과 같은 상황에서 발생한다. 

```swift
import Foundation

class Node {
    var data = ""
    var next: Node? = nil
    init(with data: String) {
        self.data = data
    }
}

struct List {
    var head: Node
    init(_ name: String) {
        head = Node(with: name)
    }
}

var a = List("JM")
var b = a

print(a.head === b.head) // true

print(b.head.data) // jm
a.head.data = "no jm"
print(b.head.data) // no jm
```

a객체를 수정 했는데, b도 함께 수정이 되어 버렸다. 단순 복사를 할 때 참조 타입의 변수의 메모리가 새로 생성되지 않고 참조값만 복사되면 이러한 일이 발생하게 된다. 이를 얕은 복사라고 한다.  swift에서는 모든 내장 타입이 struct로 되어 있다. 그래서 class를 container에 담은 채로 단순 할당을 하게 되면 얕은 복사 문제가 발생할 수 있다. 

```swift
var list = [Node(with: "A"), Node(with: "B"), Node(with: "C")]
var list2 = list
list[2].data = "D"
print(list2[2].data) // "D"
```

지금은 문제가 되지 않을 수 있지만, 프로그램의 규모가 커질 수록 이러한 오류는 찾기 어려워진다. 의도적으로 얕은 복사를 하는 것이 아니라면 깊은 복사를 통해 복사 target에게 새로운 메모리도 만들어 주어야 한다. c++같은 경우에는 연산자 오버로딩으로 해결하는데, swift에서도 이렇게 해결할 수 있는지 알아보아야겠다.

[Swift - 오퍼레이터 오버로드(Operator Overloads)](http://seorenn.blogspot.com/2014/06/swift-operator-overloads.html)

NSCopying 프로토콜을 상속받아 해결하는 방법도 있다고 한다. 하지만 결국 자기 생성자를 호출하는 로직을 내부에 만들어주어야 하는데 굳이 프로토콜을 상속받는 이유를 잘 모르겠다. 

[Apple Developer Documentation](https://developer.apple.com/documentation/objectivec/nsobject/1418807-copy)

그냥 이렇게 copy를 추가해주면 될 것 같다. 

```swift
struct List {
    var head: Node
    init(_ name: String) {
        head = Node(with: name)
    }
    func copy() -> List {
        return List(head.data)
    }
}

var a = List("JM")
//var b = a
var b = a.copy()

print(a.head === b.head)

print(b.head.data) // jm
a.head.data = "no jm"
print(b.head.data) // jm
```