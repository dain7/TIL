
# @Async

- @Transactional이 붙은 메서드에서 다른 메서드를 호출하는데, 그 메서드를 비동기로 호출해야 하는 일이 생겼다.
- 호출하는 메서드에 @Async를 붙여서 비동기로 호출해주려는데 Exception이 나면 Rollback 되어야 하는 @Transactional annotation이 잘 동작하지 않는다. 

왜일까?
1. @Transactional은 트랜잭션에 대한 정보드를 ThreadLocal로 관리하고 있다.
2. @Async는 새로운 Thread를 생성해서 메서드를 비동기로 실행시킨다.

즉, @Transactional이 붙은 메서드에 @Async가 붙은 메서드를 호출하면 @Async는 새로운 Thread를 생성하는데,
@Transactional 어노테이션은 트랜잭션을 ThreadLocal로 관리하기 때문에 기존 Transaction이 유효하지 않다.
따라서 Transaction 메서드에서 호출한 async 메서드 예외 발생 시, 호출한 메서드는 커밋된다.

