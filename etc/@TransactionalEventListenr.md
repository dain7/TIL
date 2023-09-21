# @TransactionalEventListener

- 동작하는 메서드를 트랜잭션으로 묶어서 처리하는 경우 Transaction의 상태에 따라 발생하는 이벤트를 처리해 주는 이벤트 리스너
- @EventListener의 경우 publishEvent() 메서드가 호출되는 시점에서 바로 이벤트를 publishing! -> TransactionalEventListener 사용 필요


