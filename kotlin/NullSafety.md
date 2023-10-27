# Null Safety

### ?
- ?는 널 허용 표시이다. 
- 
```kotlin
var a: String = "abc" // Regular initialization means non-nullable by default
a = null // compilation error
```

```kotlin
var b: String? = "abc" // can be set to null
b = null // ok
print(b)
```