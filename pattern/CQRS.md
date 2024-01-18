# CQRS 패턴
- 우리가 보통 이야기하는 CRUD(Create, Read, Update, Delete)에서 CUD(Command)와 R(Query)을 구분하자는 이야기다.
- 구분하는 이유는,우리가 Database로부터 데이터를 읽어오고 처리를 하게 되면 이미 그 사이에 데이터가 변경이 되었을 가능성이 높다.
- CQRS는 이런 변경 가능성을 인정하고 어차피 Read와 CUD 사이에는 delay가 존재할 수 있음을 인정하는 것이다.
- 이를 통해서 R과 CUD를 구분함으로써 얻는 이점을 설명하는 것이 CQRS패턴이다.
