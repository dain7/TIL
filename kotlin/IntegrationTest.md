# 코틀린에서 통합테스트
#### 참고자료 : https://github.com/Youhoseong/test

### Wiremock
- HTTP 기반의 api 서비스를 모킹하는 용도로 제공되는 목서버 라이브러리.
- 지정해둔 uri로 요청이 발생한 경우 목서버로 http 요청이 발생하고 미리 지정해둔 형태의 http 응답 반환

### Database
- 개발 데이터와 테스트 데이터의 분리 필요 
- H2 database를 선택 후 프로파일에 test 설정
