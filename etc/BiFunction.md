# BiFunction
```java
@FunctionalInterface
public interface BiFunction<T, U, R> {
  R apply(T t, U u);

  default <V> BiFunction<T, U, V> andThen(Function<? super R, ? extends V> after) {
    Objects.requireNonNull(after);
    return (T t, U u) -> after.apply(apply(t, u));
  }
}
```

### 제네릭 타입
T: 첫 번째 매개변수의 타입입니다.
U: 두 번째 매개변수의 타입입니다.
R: 반환 타입입니다.
