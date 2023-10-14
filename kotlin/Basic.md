# When

```kotlin
fun cases(obj: Any) {
    when (obj) {
        1 -> println("One")
        "Hello" -> println("Greeting")
        is Long -> println("Long")
        !is String -> println("Not a string")
        else -> println("Unknown")
    }
}
```
# Ranges 

- 1에서 5까지 
```kotlin
// Kotlin
for (x in 1..5) {
  println(x)
}
```
- 인덱스를 기준으로 출력
```kotlin
val list = listOf("Hello", "World", "!")
for (i in list.indices) { 
    println(list[i]) //  0, 1, 2  == for (i in 0..2)
}
```