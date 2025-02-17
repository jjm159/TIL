# 자바 기본 기억

## 참조
- 원시 타입 아니면 다 참조(heap)
- shallow copy 주의
- call by reference라는 단어
    - 직접 포인터로 관리하지 않고 감춰버리고 무조건 참조 방식으로 관리
    - 주의 하라는 의미에서 call by reference라는 말이 생긴게 아닐까?

```java

public class 참조 {
    public static void change(Student s) { // shallow cpoy
        s.name = "jm3";
    }

    public static void main(String[] args) {

        Student s = new Student("jm");

        Student[] sList = new Student[1];
        sList[0] = s;

        Student[] sList2 = sList; //  shallow copy

        change(s);

        System.out.println(sList[0]); // jm3
        System.out.println(sList2[0]); // jm3

    }
}

```

## static
### 정적 변수
- 프로그램이 생성될 때 생성되고 프로그램이 종료되면 사라지는 변수
### 정적 함수
- class에서 바로 호출 가능한 함수
- static method에서 인스턴스 변수에 접근 불가
    - 당연함
    - static method 생성 시점에 instance 참조값을 알 수 없음

## 접근 제어자
- class 내부 변수에 접근을 어디까지 허용할 건지
    - 공개의 대상이 class 내부임. class 내부를 어디까지 공개할 것이냐.
- 종류
    - private
        - 모든 클래스 외부 접근 막음
    - default
        - 같은 패키지까지 클래스 내부 접근 허용
    - protected
        - 패키지 내 접근 가능
        - 상속 관계인 경우 접근 가능(다른 패키지여도)
    - public
        - 모든 class 외부 접근 허용 
- default 접근 제어자는, 사실상 package private
- class 레벨에서는 default와 public만 사용 가능
    - private은 class 외부에서도 허용 가능하다는 건데, class에 선언한 경우 어차피 class 외부라 적용 불가 
- 캡슐화 
    - 데이터와 데이터 처리 로직을 묶어서 외부에서는 데이터 접근을 못하게 막는 것
    - 객체지향 프로그래밍의 캡슐화를 하기 위해 반드시 지원해야 하는 게 접근 제어자
- 클래스 내부의 데이터에 대한 접근을 막으면, 데이터에 대한 수정이 온전히 class 안에서만 실행됨. 그래서 해당 데이터에 대한 로직이 보장됨.

## 생성자 
### 생성자 재활용하기
```java
public Member(String name) {
    this(name, null);
}

public Member(String name, Integer age) {
    this.name = name;
    this.age = age;
}
### 기본 생성자
- 생성자가 하나도 없으면, 기본생성자를 컴파일러가 생성
- 원시타입은 0으로, 참조 타입은 null로 초기화
- 자동 생성되는 이 생성자는 class의 접근 제어자 따름
```

## 파이널
```java
package org.example;


// 얘는 상속 불가
final class Human { }

class Person {
    String name;

    public Person(String name) {
        this.name = name;
    }
}

class Math {
    public static final double PI = 3.141592; // 상수는 final로 선언해서 수정 못하게 함
}

public class 파이널 {

//    final Person p;  이렇게 선언 불가
    static final Person p = new Person("jm");

    class Test {
        final int a; // 생성자에서 초기화 하면 이렇게 가능

        public Test(int a) {
            this.a = a;
        }
    }

//    final int b; // 이렇게 선언 불가
    final int b = 1; // 한번만 할당이 가능하니까!

//    static final int c; // 이렇게 선언 불가
    static final int c = 0;

    static void test(final int a) {
        // a = 2; // 이게 안됨
    }
    public static void main(String[] args) {

        final int a = 1;
        // a = 2; // 이게 안됨
        // 최초 한번만 할당 가능함

//        p = new Person("jm2"); 이거 안됨. final이라서
        p.name = "jm2"; // 이건 되고, p에 재할당은 안됨.

    }
}

```
