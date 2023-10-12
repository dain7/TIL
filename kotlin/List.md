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

### groupBy
- groupBy 함수는 Map<K, List<V>> 형태의 결과를 반환한다.
- 키와 키에 해당하는 요소들을 리스트로 묶은 맵을 반환한다.
- 이 함수는 컬렉션의 각 요소에 대해 키를 추출하여 그룹화하고, 각 그룹은 해당 키와 일치하는 요소들의 리스트로 표현된다.
```kotlin
val words = listOf("apple", "banana", "cherry", "date", "elderberry")

val groups = words.groupBy { it.length }

println(groups) // 출력 : {5=[apple], 6=[banana, cherry], 4=[date], 10=[elderberry]}
```
