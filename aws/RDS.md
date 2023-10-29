# RDS (Relational Database Service)

- 간단히 말하면 관계형 데이터베이스를 제공하는 AWS의 서비스다. 
- 유저가 사용하기 쉽도록 인프라 등을 자동화 시켜주고 유저들은 엔드포인트로 접속할 수 있는 데이터 베이스를 제공받는다.

### 특징

- 관계형 데이터베이스를 제공하는 서비스
- 가상머신 위에서 동작
  - 시스템에 직접 로그인 불가능 
- CloudWatch와 연동
  - DB 인스턴스의 모니터링
  - DB에서 발생하는 여러 로그를 확인 가능
- 내부에서는 EC2 사용
  - VPC(고객 전용 사설 네트워크) 안에서 동작 
  - Storage는 EBS 활용
- Parameter Group
  - Root 유저만 설정 가능한 DB의 설정값들을 모아 그룹화한 개념

### RDS 인증 방법
##### 전통적인 유저 / 패스워드방식
AWS Secret 매니저 (RDS를 비롯해 DB, 여러 인증을 관리해주는 서비스, 자동으로 일정 주기마다 패스워드 변경, 다른 서비스들이 RDS 혹은 서비스들을 접근할 때 암호를 하드코딩 할 필요 없게 만들어줌)와 연동하여 자동 로테이션 가능
##### IAM DB 인증
데이터베이스를 IAM 유저 크레덴셜 / Role을 통해 관리 가능
##### Kerberos 
MS Active Directory 안에 들어있는 프로토콜

### RDS에서 제공하는 DB 엔진
- 라이센스 비용 X
  - my sql, postresql, maria db
- 라이센스 비용
  - my sql, oracle
