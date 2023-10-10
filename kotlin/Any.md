# Any

- 코틀린에서 모든 타입이 상속받는 최상위 타입
- 자바로 디컴파일 해보면 Object 타입으로 변환된다.

### Object vs Any

- 자바에서는 참조 타입만 Object를 최상위 클래스로 상속 받는다. 코틀린은 원시타입과 참조타입을 구별하지 않기 때문에 Any가 모든 타입의 조상 클래스가 된다.
- wait(), notify(), notifyAll() 처럼 쓰레드를 제어하는 메서드는 Any에 없다.
- 자바의 object는 toString, equals, hasCode, wait, notify를 지원하지만 Any는 toString, equals, hasCode만 지원한다.

### is

- 자바의 instance of 연산자와 같은 역할
- 리턴 값은 true, false

```kotlin
fun isInt(var1 : Any) : Boolean{
    return (var1 is Int)
}
```
