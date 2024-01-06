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

### example
```kotlin
import kotlin.reflect.full.memberProperties

data class Example(val name: String?, val age: Int?)

fun main() {
    val example = Example(null, 25) // object
    val fieldName = "name" // 필드명

    val property = Example::class.memberProperties.firstOrNull { it.name == fieldName } // 해당 프로퍼티의 필드명과 staitc 필드명으로 찾기

    if (property != null) {
        val value = property.get(example) // 해당 값 가져오기
        if (value == null) {
            println("$fieldName is null")
        } else {
            println("$fieldName: $value")
        }
    } else {
        println("Property with name $fieldName not found")
    }
}
```