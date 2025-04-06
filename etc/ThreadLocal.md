# ThreadLocal
- 스레드 단위로 로컬 변수를 사용할 수 있다.
- 그러나 잘못 사용하는 경우 부작용이 발생할 수 있어 다른 스레드와 변수가 공유되지 않도록 주의해야 한다.

### set, get
```java
public void set(T value) {
    // 스레드 확인
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        map.set(this, value);
    } else {
        createMap(t, value); 
    }
}

public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}

ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}

void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

### withinitial
- 스레드 로컬 변수를 생성하면서 특정 값으로 초기화하는 메서드
```java
public static <S> ThreadLocal<S> withInitial(Supplier<? extends S> supplier) {
    return new SuppliedThreadLocal<>(supplier);
}
```

### remove
- 스레드 로컬 변수 값을 삭제하는 메서드
- 스레드 풀을 사용한다면 remove를 명시적으로 호출해야 함 (스레드 풀을 통해 스레드가 재사용 되기 때문에!)
- 스레드가 재활용되면서 이전에 설정했던 스레드 로컬 정보가 남아있을 수 있기 때문에
```java
public void remove() {
     ThreadLocalMap m = getMap(Thread.currentThread());
     if (m != null)
         m.remove(this);
 }
```