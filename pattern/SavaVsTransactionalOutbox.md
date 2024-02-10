# Saga vs Transactional Outbox 

### SAGA
- 각 서비스를 업데이트 하고 메세지나 이벤트를 게시해서 다음 트랜잭션 실행을 트리거한다.
- 연속적으로 로컬 트랜잭션을 실행하다가 중간에 문제가 생겨 로컬 트랜잭션이 실패하면 이전의 로컬 트랜잭션에 의해 변경했던 사항을 취소하는 보상 트랜잭션 실행
- 
- 
참조: https://devocean.sk.com/blog/techBoardDetail.do?ID=165445