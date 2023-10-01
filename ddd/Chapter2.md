# 아키텍처 개요 

### 아키텍처를 설계할 때 출현하는 전형적인 영역
- 표현(컨트롤러) : 사용자의 요청을 받아 응용 영역에 전달하고 처리 결과를 다시 사용자에게 보여줌.
- 응용(서비스) : 사용자에게 제공할 기능을 구현. 도메인 모델에 로직 수행을 위임.
- 도메인 : 도메인 모델을 구현.
- 인프라스트럭처 : RDBMS 연동, 메세징 큐 송수신, DB 연동 등 실제 구현을 다룸.

### 계층 구조 아키텍처

전형적인 계층 구조상의 의존관계는
- 서비스 -> 엔티티 의존
- 서비스 -> DB 모듈 의존
- 엔티티 -> 룰 엔진 의존

응용 영역과 도메인 영역은 인프라스트럭처의 기능을 사용하므로 이런 계층 구조를 사요하는 것은 직관적으로 이해 가능! </br>
그러나, 이런 경우의 문제점은 

- 서비스 테스트시 룰 엔진 또한 테스트 설정 필요 
- 기능 확장의 어려움 발생 

이를 위해서는 <b>DIP</b>의 적용이 필요하다!

### DIP

가격 할인 계산을 하려면 다음과 같은 룰을 실행해야 한다 </br>

[고수준]
- 가격 할인 계산 (CalculateDiscountService)
  - 고객 정보를 구한다
  - 룰을 이용해서 할인 금액을 구한다. 

[저수준]
- 고객 정보를 구한다 -> RDBMS에서 JPA로 구한다.
- 룰을 이용해서 할인 금액을 구한다 -> Drools로 룰을 적용한다.

<b>고수준 모듈</b>은 의미있는 단일 기능을 제공하는 모듈. 고수준 모듈의 기능을 구현하려면 여러 하위 기능 필요. </br>
<b>저수준 모듈</b>은 하위 기능을 실제로 구현한 것. </br>

고수준 모듈이 제대로 동작하려면 저수준 모듈을 사용해야 한다. 그런데 고수준 모듈이 저수준 모듈을 사용하면 앞서 계층 구조 아키텍처에서 언급했던 두 가지 문제가 발생한다. </br>
<b>DIP는 이 문제를 해결하기 위해 저수준 모듈이 고수준 모듈에 의존하도록 바꾼다.</b>

##### <기존 서비스>
```java
public class CalculateDiscountService {
    private DroolsRuleEngine ruleEngine;
    
    public CalculateDiscountService() {
        ruleEngine = new DroolsRuleEngine();
    }
    
    public Money calculateDiscount(OrderLine orderLines, String customerId) {
        Customer customer = findCustomer(customerId);
        
        MutableMoney money = new MutableMoney(0);
        List<?> facts = Arrays.asList(customer, money);
        ,,,
        return money.toImmutableMoney();
    }
}
```

##### DIP 적용 
```java
public interface RuleDiscounter {
    public Money applyRules(Customer customer, List<OrderLine> orderLines);
}
```
##### <기존 서비스>
```java
public class CalculateDiscountService {
    private RuleDiscounter ruleDiscounter;
    
    public CalculateDiscountService(RuleDiscounter ruleDiscounter) {
        this.ruleDiscounter = ruleDiscounter;
    }
    
    public Money calculateDiscount(OrderLine orderLines, String customerId) {
        Customer customer = findCustomer(customerId);

        return ruleDiscounter.applyRules(customer, orderLines)
    }
}
```
- 룰 적용을 구현한 클래스는 RuleDiscounter 인터페이스를 상속받아 구현한다.

CalculateDiscountService는 <인터페이스>RuleDiscounter에 의존한다. </br>
DroolsRuleDiscounter는 RuleDiscounter를 구현한다. </br>

DIP를 적용하면 저수준 모듈이 고수준 모듈에 의존하게 된다. 고수준 모듈이 저수준 모듈을 사용하려면 고수준 모듈이 저수준 모듈에 의존해야 하는데, 반대로 저수준 모듈이 고수준 모듈에 의존한다고 해서 이를 DIP라고 부른다. </br>
DIP를 적용하면 1. 구현 교체가 어렵다, 2. 테스트가 어렵다 문제를 해소할 수 있다. </br>

먼저 구현 기술 교체문제를 보면, 고수준 모듈은 더이상 저수준 모듈에 의존하지 않고 구현을 추상화한 인터페이스에 의존한다.

```java
// 사용할 저수준 객체 생성 
RuleDiscounter ruleDiscounter = new DroolsRuleDiscounter();

// 생성자 방식으로 주입 
CalculateDiscountService disService = new CalulateDiscountService(ruleDiscounter);
```
```java
// 사용할 저수준 객체 변경
RuleDiscounter ruleDiscounter = new SimpleRuleDiscounter();

// 사용할 저수준 모듈을 변경해도 고수준 모듈을 수정할 필요가 없다.
CalculateDiscountService disService = new CalulateDiscountService(ruleDiscounter);
```

의존 주입을 지원하는 스프링과 같은 프레임워크를 사용하면 설정 코드를 수정해서 쉽게 구현체를 변경할 수 있다.</br>

```java
public class CalculateDiscountService {
    private CustomerRepository customerRepository;
    private RuleDiscounter ruleDiscounter;
  ...
}
```

테스트 또한 CaculateDiscoutService가 저수준 모듈에 직접 의존했다면 저수준 모듈이 만들어지기 전까지 테스트를 할 수가 없었겠지만 CustomerRepository와 RuleDiscounter는 인터페이스이므로 대용객체를 사용해서 테스트 진행 가능! </br>

### DIP 주의사항
DIP를 잘못 생각하면 단순히 인터페이스와 구현 클래스를 분리하는 정도로 받아들일 수 있다. </br>
DIP의 핵심은 고수준 모듈이 저수준 모듈에 의존하지 않도록 하기 위함이다!!!

### 도메인 영역의 주요 구성요소 

- 엔티티 : 고유의 식별자를 갖는 객체로 자신의 라이프사이클을 갖는다. 
- 밸류 : 개념적으로 하나인 도메인 객체의 속성을 표현할 때 사용된다.
- 애그리거트 : 관련된 엔티티와 밸류 객체를 개념적으로 하나로 묶은 것이다. 예를 들어 주문과 관련된 Order 객체, OrderLine 밸류, Orderer 밸류 객체를 '주문' 애그리거트로 묶을 수 있다.
- 레포지터리 : 도메인 모델의 영속성을 처리한다. 
- 도메인 서비스 : 특정 엔티티에 속하지 않은 도메인 로직을 제공한다. 
