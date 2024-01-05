# Member Properties

### Reflection
- 리플렉션이랑 런타임에 프로그램의 클래스를 조사하기 위해 사용되는 기술.

### 리플렉션 접근
- 코틀린에서 리플렉션을 이용하기 위해서 사용하는 객체를 레퍼런스 객체라고 한다.
- 레퍼런스 객체를 만드는 방법
  - 1) 인스턴스를 활용한다.
  - 2) 클래스를 이용한다.

```kotlin
// 인스턴스 활용 
val sabarada = Sabarada(5)
val kClass: KClass<out Sabarada> = sabarada::class

// 클래스 이용
val kClass: KClass<Sabarada> = Sabarada
```

### 리플렉션을 이용한 변수 접근

```kotlin
val kClass: KClass<Sabarada> = Sabarada::class
val primaryConstructor = kClass.primaryConstructor
val instance = primaryConstructor?.call(5)

val immutableSecret = kClass.memberProperties.find { it.name == "immutableSecret" }
immutableSecret?.isAccessible = true
println("immutableSecret = ${immutableSecret?.get(instance!!)}") // immutableSecret = 2
```