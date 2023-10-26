# for
- @을 이용하여 라벨링 
```kotlin
// @을 만나면 가장 바깥쪽에 있는 for문 탈출 
loop@ for (i in 1..100) {
    for (j in 1..100) {
        if (...) break@loop
    }
}
```

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

// 모든 케이스가 커버되면 else 필요없음 (ex. Enum 사용,..)
enum class Bit {
    ZERO, ONE
}

val numericValue = when (getRandomBit()) {
    Bit.ZERO -> 0
    Bit.ONE -> 1
    // 'else' is not required because all cases are covered
}
```

# Ranges

- 1에서 5까지

```kotlin
for (x in 1..5) {
  println(x)
}

// when 사용
when (x) {
    in 1..10 -> print("x is in the range")
    in validNumbers -> print("x is valid")
    !in 10..20 -> print("x is outside the range")
    else -> print("none of the above")
}
```

- 인덱스를 기준으로 출력

```kotlin
val list = listOf("Hello", "World", "!")
for (i in list.indices) {
    println(list[i]) //  0, 1, 2  == for (i in 0..2)
}
```
