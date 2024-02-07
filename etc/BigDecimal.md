# BigDecimal

- Java에서 숫자를 정밀하게 저장하고 표현할 수 있는 유일한 방법!
- 돈과 소수점을 다룬다면 필수!

### double의 문제점

- double 타입은 내부적으로 수를 저장할 때 이진수의 근사치를 저장한다.
- 저장된 수를 다시 십진수로 표현하면서 아래와 같은 문제가 발생한다.

```java
double a = 10.0000;
double b = 3.0000;

// 기대값: 13
// 실제값: 13.000001999999999
a + b;
```

### Big decimal 기본용어

- precision: 숫자를 구성하는 전체 자리수. 왼쪽부터 0이 아닌 수가 시작하는 위치부터 오른쪽부터 0이 아닌 수로 끝나는 위치까지의 총 자리수다.
- scale: 소수점 첫째자리부터 오른쪽부터 0이 아닌수로 끝나는 위치까지의 총 소수점 자리수

### 기본상수

```java
// 흔히 쓰이는 값은 상수로 정의
// 0
BigDecimal.ZERO

// 1
BigDecimal.ONE

// 10
BigDecimal.TEN
```

### 초기화

```java
// double 타입을 그대로 초기화하면 기대값과 다른 값을 가진다.
// 0.01000000000000000020816681711721685132943093776702880859375
new BigDecimal(0.01);

// 문자열로 초기화하면 정상 인식
// 0.01
new BigDecimal("0.01");

// 위와 동일한 결과, double#toString을 이용하여 문자열로 초기화
// 0.01
BigDecimal.valueOf(0.01);
```

### 소수점 처리

```java
// 소수점 이하를 절사한다.
// 1
new BigDecimal("1.1234567890").setScale(0, RoundingMode.FLOOR);

// 소수점 이하를 절사하고 1을 증가시킨다.
// 2
new BigDecimal("1.1234567890").setScale(0, RoundingMode.CEILING);
// 음수에서는 소수점 이하만 절사한다.
// -1
new BigDecimal("-1.1234567890").setScale(0, RoundingMode.CEILING);

// 소수점 자리수에서 오른쪽의 0 부분을 제거한 값을 반환한다.
// 0.9999
new BigDecimal("0.99990").stripTrailingZeros();

// 소수점 자리수를 재정의한다.
// 원래 소수점 자리수보다 작은 자리수의 소수점을 설정하면 예외가 발생한다.
// java.lang.ArithmeticException: Rounding necessary
new BigDecimal("0.1234").setScale(3);

// 반올림 정책을 명시하면 예외가 발생하지 않는다.
// 0.123
new BigDecimal("0.1234").setScale(3, RoundingMode.HALF_EVEN);

// 소수점을 남기지 않고 반올림한다.
// 0
new BigDecimal("0.1234").setScale(0, RoundingMode.HALF_EVEN);
// 1
new BigDecimal("0.9876").setScale(0, RoundingMode.HALF_EVEN);
```
