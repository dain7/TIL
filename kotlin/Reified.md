# Reified
- Reified 키워드는 Generics로 inline function에서 사용되며, Runtime에 정보를 알고 싶을 때 사용한다.
- Generics는 아래와 같이 T와 같은 문자로 타입을 표현한다.

```kotlin
fun <T> function(argument: T)
```

- Generics 코드를 컴파일 할 때 컴파일러는 T가 어떤 타입인지 알고 있다. 
- 하지만 컴파일 하면서 타입 정보를 제거하여 Runtime에는 T가 어떤 타입인지 모른다.

<b> Reified 키워드를 사용하면 Generic function에서 Runtime에 타입 정보를 알 수 있다 </b>
<b> 하지만 inline function과 함께 사용할 때만 가능하다. </b>

- Reified는 다음과 같이 정의가 가능하다.


```kotlin
fun <T> function(argument: T)
```

- T::class처럼 클래스의 타입 정보를 얻을 수 있다.
- 그래서 타입 정보를 인자로 넘기지 않아도 Runtime에 타입 정보를 알 수 있다.
```kotlin
inline fun <reified T> printGenerics(value: T) {
    when (T::class) {
        String::class -> {
            println("String : $value")
        }
        Int::class -> {
            println("Int : $value")
        }
    }
}

printGenerics1("print generics function")
printGenerics1(1000)
```