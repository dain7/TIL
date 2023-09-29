# 도메인 모델 시작

### 도메인

책을 구매할 때 이용하는 온라인 서점. 어떤 책이 나왔는지 검색하고, 책의 목차와 서평을 본다. 장바구니에 담아두기도 하고, 바로 구매도 할 수 있다 </br>
쿠폰을 사용하기도 하고, 결제수단으로 신용카드도 있고, 가상계좌를 사용할 수 도 있다. 구매 후 언제쯤 책을 받아볼 수 있는지 궁금해서 배송추적 기능을 사용한다.</br>
개발자 입장에서 온라인 서점은 <b>구현해야 할 소프트웨어의 대상</b>이 된다. 온라인 서점 소프트웨어는 온라인으로 책을 판매하는데 필요한 상품 조회, 구매, 결제, 배송 추적 등의 기능을 제공해야한다. </br>
이때 온라인 서점은 소프트웨어로 해결하고자 하는 문제 영역 <b>도메인</b>에 해당한다. </br>

### 도메인 모델

기본적으로 도메인 모델은 특정 도메인을 개념적으로 표현한 것이다. </br>
도메인을 객체 기반으로 모델링을 하거나, 상태 다이어그램을 이용해 상태 모델링을 할 수 있다. </br>
중요한 것은 도메인 자체를 이해하는데 도움을 줘야 한다는 것이다. </br>

### 도메인 모델 패턴

일반적인 애플리케이션 아키텍처는 네 개의 계층으로 구성된다.

- 사용자 인터페이스 : 사용자의 요청을 처리하고 사용자에게 정보를 보여준다.
- 응용 : 사용자가 요청한 기능을 실행한다. 업무 로직을 직접 구현하지 않으며 도메인 계층을 조합해서 기능을 실행한다.
- 도메인 : 시스템이 제공할 도메인 규칙을 구현한다.
- 인프라 스트럭쳐 : 데이터베이스나 메시징 시스템과 같은 외부 시스템과의 연동을 처리한다.

도메인 계층은 도메인의 핵심 규칙을 구혀한다. </br>
주문 도메인의 경우 '출고 저에 배송지를 변경할 수 있다'는 규칙과 '주문 취소는 배송전에만 할 수 있다'는 규칙을 구현한 코드가 도메인 계층에 위치하게 된다. </br>
이런 도메인 규칙을 객체 지향 기법으로 구현하는 패턴이 도메인 모델 패턴이다.

```java
public class Order {
    private OrderState state;
    private ShippingInfo shippingInfo;

    public void changeShippingInfo(ShippingInfo newShippingInfo) {
        if (!state.isShippingChangeable()) {
            throw new IllegalStateException("can't change shipping in " + state);
        }
        this.shippingInfo = newShippingInfo;
    }

    public void changeShipped() {
        this.state = OrderState.SHIPPED;
    }
}

public enum OrderState {
    PAYMENT_WAITING {
        public boolean isShippingChangeable() {
            return true;
        }
    },
    PEPARING {
        public boolean isShippingChagaeble() {
            return true;
        }
    },
    SHIPPED, DELIVERING, DELIVERY_COMPLETED;

    public boolean isShippingChaneable() {
        return false;
    }
}
```

- OrderState는 주문 대기 중이거나, 상품 준비 중에는 배송지를 변경할 수 있다는 도메인 규칙을 구현.
- Order 클래스는 OrderState의 isShippingChangeable() 메서드를 이용해서 변경 가능 여부를 확인한 후 변경 가능한 경우에만 배송지를 변경한다.
  </br>
  큰 틀에서 보면 OrderState는 Order에 속한 데이터이므로 배송지 정보 변경 가능 여부를 판단하는 코드를 Order로 이동할 수도 있다.
  다음은 Order 크래스에서 판단하도록 수정한 코드를 보여주고 있다.

```java
public class Order {
    private OrderState state;
    private ShippingInfo shippingInfo;

    public void changeShippingInfo(ShippingInfo newShippingInfo) {
        if (!isShippingChangeable()) {
            throw new IllegalStateException("can't change shipping in " + state);
        }
        this.shippingInfo = newShippingInfo;
    }

    public boolean isShippingChangeable() {
        return state = OrderState.PAYMENT_WWAITING || state == OrderState.WATING;
    }
}
```

배송지 변경 가능 여부를 판단하는 기능이 Order에 있든, OrderState에 있든 중요한 점은 주문과 관련된 중요 업무 규칙을 주문 도메인 모델인 Order나 OrderState에서 구현한다는 점이다. 핵심 규칙을 구현한 코드는 도메인 모델에만 위치하기 때문에 규칙이 바뀌거나 규칙을 확장해야 할 때 다른 코드에 영향을 덜 주고 변경 내역을 모델에 반영할 수 있게 된다 .

### 도메인 모델 도출

도메인을 모델링 할 때 기본이 되는 작업은 모델을 구성하는 핵심 구성요소, 규칙, 기능을 찾는 것이다. </br>
이 과정은 요구사항에서 출발한다. 주문 도메인과 관련된 몇가지 요구사항을 보자 </br>

- 최소 한 종류 이상의 상품을 주문해야 한다.
- 한 상품을 한 개 이상 주문할 수 있다.
- 총 주문 금액은 각 상품이 구매 가격 합을 모두 더한 금액이다.
- 각 상품의 구매 가격 합은 상품 가격에 구매 개수를 곱한 값이다.
- 주문할 때 배송지 정보를 반드시 지정해야 한다.
- 배송지 정보는 받는 사람 이름, 전화번호, 주소로 구성된다.
- 출고를 하면 배송지 정보를 변경할 수 없다.
- 출고 전에 주문을 취소할 수 있다.
- 고객이 결제를 완료하기 전에는 상품을 준비하지 않는다.

이 요구사항에서 알 수 있는건 주문은 다음과 같은 기능을 제공한다는 것이다.

- 출고 상태로 변경하기
- 배송지 정보 변경하기
- 주문 취소하기
- 결제 완료로 변경하기

```java
public class Order {
    public void changeShipped() { ... }
    public void changeShippingInfo() { ... }
    public void cancel() { ... }
    public void completePayment() { ... }
}
```

다음 요구사항은 주문 항목이 어떤 데이터로 구성되는지 알려준다.

- 한 상품을 한 개 이상 주문할 수 있다.
- 각 상품의 구매 가격 합은 상품 가격에 구매 개수를 곱한 값이다.

```java
public class OrderLine {
    private Product product;
    private int price;
    private int quantity;
    private int amounts;

    public OrderLine(Product product, int price, int quantity) {
        this.product = product;
        this.price = price;
        this.quantity = quantity;
        this.amounts = calculateAmounts();
    }

    private int calculateAmounts() {
        return price * quantity;
    }

    public int getAmounts() { ... }
}
```

다음 요구사항은 Order와 OrderLine과의 관계를 알려준다.
- 최소 한 종류 이상의 상품을 주문해야 한다.
- 총 주문 금액은 각 상품의 구매 가격 합을 모두 더한 금액이다.

```java
public class Order {
    private List<OrderLine> orderLines;
    private int totalAmounts;
    
    public Order(List<OrderLine> orderLines) {
        setOrderLines(orderLines);
    }
    
    private void setOrderLines(List<OrderLine> orderLines) {
        verifyAtLeastOneOrMoreOrderLines(orderLines);
        this.orderLines = orderLines;
        calcualteTotalAmounts();
    }
    
    private void verifyAtLeastOneOrMoreOrderLines(List<OrderLine> orderLines) {
        if (orderLines == null || orderLines.isEmpty()) {
            throw new IllegalArgumentException("no orderline");
        }
    }
    
  ... // 
}
```

배송지 정보는 이름, 전화번호, 주소 데이터를 가지므로 다음과 같이 정의한다.

```java
public class ShippingInfo {
    private String receiverName;
    private String receiverPhonNumber;
    private String shippingAddress1;
    private String shippingAddress2;
    private String shippingZipcode;
  ... //
}
```

도메인을 구현하다보면 특정 조건이나 상태에 따라 제약이나 규칙이 달리 적용되는 경우가 많다.
- 출고를 하면 배송지 정보를 변경할 수 없다.
- 출고 전에 주문을 취소할 수 있다. 

이 요구사항은 출고 상태가 되기 전과 후의 제약사하을 기술하고 있다. </br>
다음 요구사항도 상태와 관련이 있다. 

- 고객이 결제를 완료하기 전에는 상품을 준비하지 않는다.

```java
public enum OrderState {
    PAYMENT_WAITING, PREPATING, SHIPPED, DELIVERING, DELIVERY_COMPLETED, CANCELED
}
```

이런 식으로 요구사항에 ㄷ맞춰 도메인 모델을 점진적으로 만들면 된다!

### 엔티티

엔티티의 가장 큰 특징은 식별자를 갖는다는 것이다. </br>
주문에서 배송지 주소가 바뀌거나 상태가 바뀌더라도 주문번호가 바뀌지 않는 것처럼 엔티티의 식별자는 바뀌지 않는다. </br>
엔티티의 식별자는 바뀌지 ㅇ낳고 고유하기 때문에 두 엔티티 객체의 식별자가 같으면 두 엔티티는 같다고 판단할 수 있다. 
 
### 엔티티의 식별자 생성

- 특정 규칙에 따라 생성
- UUID 사용
- 값을 직접 입력
- 일련번호 사용 

### 밸류 타입

ShippingInfo 클래스는 받는 사람과 주소에 대한 데이터를 갖고 있다.
```java
public class ShippingInfo {
    // 받는 사람
    private String receiverName;
    private String receiverPhoneNumber;
    
    // 주소
    private String shippingAddress1;
    private String shippingAddress2;
    
  ... // 
}
```

ShippingInfo 클래스의 receiverName 필드와 receiverPhoneNumber 필드는 서로 다른 두 데이터를 담고 있지만 두 필드는 개념적으로 받는 사람을 의미한다.

```java
public class Receiver {
    private String name;
    private String phoneNumber;
    
    public Receiver(String name, String phoneNumber) {
        this.name = name;
        this.phoneNumber = phoneNumber
    }
    
    public String getName() {
        return name;
    }
    
    public String getPhoneNumber() {
        return phoneNumber;
    }
}
```

Receiver는 '받는 사람'이라는 도메인 개념을 표현한다. 

```java
public class ShippingInfo {
    private Receiver receiver;
    private Address address;
}
```
밸류 타입을 사용할 때의 또 다른 장점은 밸류 타입을 위한 기능을 추가할 수도 있다는 것이다. </br>
예를 들어, Money 타입은 다음과 같이 돈 계산을 위한 기능을 추가할 수 있다. 

```java
public class Money {
    private int value;
    
    public Money add(Money money) {
        return new Money(this.value + money.value);
    }
    
    public Money multiply(int multiplier) {
        return new Money(value * multiplier);
    }
}
```

밸류 객체의 데이터를 변경할 때는 기존 데이터를 변경하기보다는 변경한 데이터를 갖는 새로운 밸류 객체를 생성하는 방식을 선호한다. </br>
Money처럼 데이터 변경 기능을 제공하지 않는 타입을 불변이라고 표현한다. 불변으로 구현하는 이유는 보다 안전한 코드를 작성할 수 있기 때문이다. 

