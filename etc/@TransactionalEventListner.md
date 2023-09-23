# @TransactionalEventListener

- 동작하는 메서드를 트랜잭션으로 묶어서 처리하는 경우 Transaction의 상태에 따라 발생하는 이벤트를 처리해 주는 이벤트 리스너
- @EventListener의 경우 publishEvent() 메서드가 호출되는 시점에서 바로 이벤트를 publishing! -> TransactionalEventListener 사용 필요

## 예시
- 다음과 같은 상황이 있다.
- 1번과 2번이 정상적으로 마무리되고 3번에서 예외처리가 발생하면? -> 1,3번은 실패지만 2번은 rollback이 이루어지지 않음.
- 그래서 나온게 TransactionalEventListener -> 이벤트의 발생을 트랜잭션의 종료를 기준으로 삼는다!
```java
@Transactional
public void function() {

    reviewRepository.save() // 1. A 저장

    applicationEventPublisher.publishEvent(); // 2. A에 의한 이벤트 발생

    userRepository.save() // 3. B 저장

}
```

## 옵션
1. After Commit (기본값) : 트랜잭션이 성공적으로 마무리 됐을 때 이벤트 실행
2. After Rollback : 트랜잭션이 롤백 됐을 때 이벤트 실행
3. After Completion : 트랜잭션이 마무리 됐을 때 이벤트 실행
4. Before Commit : 트랜잭션의 커밋 전에 이벤트 실행
