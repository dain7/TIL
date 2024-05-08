# Functional Interface
- 함수형 인터페이스 
  - 1개의 추상 메소드를 갖는 인터페이스

### 기본 제공
#### Function
```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}
```
- T 타입 인자를 받아서 R 타입을 리턴한다