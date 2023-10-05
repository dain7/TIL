# List

### listOf 
- 리스트를 만들 때 사용하는 함수. 읽기 전용 모드이다. 
- .add 는 가능하나 name이 재할당 되지 않는다.

```kotlin
val name = listOf("a", "b", "c")
```

### mutableListOf
- 수정 가능한 리스트 생성

```kotlin
var name = mutableListOf("a", "b", "c")
```

### filterNotNull
- null을 제외한 element 출력

```kotlin
name.filterNotNull()
```

### +, -

- +,- 를 사용하여 엘리먼트 추가 및 삭제 가능

```kotlin
var name = mutableListOf("a", "b", "c") - "a"
```

