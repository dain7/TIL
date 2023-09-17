# Inline-Functions 
- 인라인(inline) 키워드는 자바에서는 제공하지 않는 코틀린만의 키워드.
- 코틀린에서 고차함수(함수를 인자로 전달하거나 함수를 리턴)를 사용하면 <b>패널티가 발생</b>한다고 나와있다.
- 패널티란 추가적인 메모리 할당 및 함수 호출로 Runtime overhead가 발생한다는 것으로, 람다를 사용하면 각 함수는 개체로 변환되어 메모리 할당과 가상 호출 단계를 거치는데 여기서 런타임 오버헤드가 발생한다.
- 인라인 함수는 내부적으로 함수 내용을 호출되는 위치에 복사하며, 런타임 오버헤드를 줄여준다.

# 객체로 변환되는 오버헤드란?
- 아래와 같은 고차함수를 컴파일 하면 자바 파일이 생성된다.
```kotlin
// 앞으로 사용할 메소드(함수)
fun doSomethingElse(lambda: () -> Unit) {
    println("Doing something else")
    lambda()
}
```
- 아래 컴파일된 자바 코드는 functional interface인 function 객체를 파라미터로 받고 invoke 메서드를 실행한다.
```java
public static final void doSomethingElse(Function0 lambda) {
    System.out.println("Doing something else");
    lambda.invoke();
}
```
- 위에서 선언한 메소드를 이용해 다음과 같은 메소드를 사용해보자.
- doSomethingElse를 실행하기 전 출력문을 실행 후 함수를 호출하며 파라미터로 람다식을 넣었다.
```java
// 1번 메소드의 자바 컴파일 코드
public static final void doSomething() {
    System.out.println("Before lambda");
    
    doSomethingElse(new Function() {   // "<---여기가 문제임 "
            public final void invoke() {
            System.out.println("Inside lambda");
        }
    });
    
    System.out.println("After lambda");
}
```
- 여기서 문제점은 파라미터로 매번 새로운 new Function() 객체를 만들어 넣어준다는 것이다.
- 이렇게 의미없이 객체로 변환되는 코드가 바로 객체로 변환되는 오버헤드이자 패널티이다.

# inline-functions로 오버헤드 해결하기
```kotlin
// inline을 붙인 2번 메소드
inline fun doSomethingElse(lambda: () -> Unit) {
   println("Doing something else")
   lambda()
}
```
- 위의 코드를 자바코드로 컴파일 하면 다음과 같다.
```java
// 2번 메소드의 자바 컴파일 코드
public static final void doSomething() {
    System.out.println("Before lambda");
    System.out.println("Doing something else");
    System.out.println("Inside lambda");
    System.out.println("After lambda");
}
```
- 새로운 객체를 생성하는 부분이 사라지고, print문으로 변경되었다.

