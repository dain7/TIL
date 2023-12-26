# 사가 패턴

- 로컬 트랜잭션들이 순차적으로 업데이트를 수행해 나가는 패턴을 의미한다.
- 각각 마이크로 서비스에서 책임지고 트랜잭션을 처리하는데 문제 발생 -> 보상 이벤트를 통해 보상 트랜잭션을 발행하여 이전 서비스들에게 롤백하자
- MSA는 물리적으로 DB가 분뤼되어 있어 단일 트랜잭션으로 커밋, 롤백 처리가 불가능하다 -> MSA는 비즈니스 로직으로 커밋, 롤백 등 트랜잭션에 대한 책임을 가지도록 구현 필요

### Orchestration based SAGA pattern

- 분산 트랜잭션을 책임지는 별도의 중계자가 존재 (분산 트랜잭션의 중앙 집중화)
- 중계자는 사가 인스턴스를 발급하여 각 작업의 상태를 저장 및 해석하며, 보상 트랜잭션을 사용하여 오류 복구를 처리한다.
- 장점: 서비스 간의 복잡성이 줄어들면서 구현 및 테스트가 쉬워짐
- 단점: 트랜잭션의 현재 상태를 manager가 알고 있으므로 롤백하기 쉬움

### Choreography base SAGA pattern

- 분산 트랜잭션을 책임지는 중계자가 존재하지 않는 방식
- 순서
  - 로컬 트랜잭션을 처리하고 다음 서비스에게 이벤트 전달
  - 성공, 실패를 큐로 응답으로 넣어준다.
  - 실패시 보상 트랜잭션 발행
- 장점: 구성하기 편함
- 단점: 트랜잭션의 현재 상태 파악이 어려움

### Saga pattern 플로우

1. Begin Order: When a new order is initiated, create a new saga instance and store it in a database. The saga instance should include all the data required to perform the order process, such as the user’s information and the order details. The initial step in the saga is to create a new order in the database.

2. Process Payment: The next step in the saga is to process the payment. If the payment is successful, update the saga instance to mark the order as paid in the database. If the payment fails, the saga should trigger a compensation step to cancel the order and roll back any previous steps.

3. Check Inventory: After the payment has been processed successfully, the next step is to check the inventory to see if all the items in the order are available. If any items are out of stock, the saga should trigger a compensation step to cancel the order and roll back any previous steps.
