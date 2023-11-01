# 코틀린에서 통합테스트
#### 참고자료 : https://github.com/Youhoseong/test

### Wiremock
- HTTP 기반의 api 서비스를 모킹하는 용도로 제공되는 목서버 라이브러리.
- 지정해둔 uri로 요청이 발생한 경우 목서버로 http 요청이 발생하고 미리 지정해둔 형태의 http 응답 반환
- 서버로부터 받지 못하는 상황을 테스트 할 수 있다. 

```yaml
testImplementation("org.springframework.cloud:spring-cloud-contract-wiremock")
```

```kotlin
@ActiveProfiles("wiremock")
@AutoConfigureWireMock(port = 0) // 랜덤포트 상에서 실행되게 
@AutoConfigureTestFeign
@SpringBootTest(
    classes = [FeignClientConfiguration::class]
)
abstract class FeignWireMockTest {
    @Autowired
    protected lateinit var 
```

```kotlin
@Test
fun `503이면 IllegalStateException을 던진다`() {
    // given
    stubFor(
        get(anyUrl())
            .willReturn(serviceUnavailable())
    )
    // when
    val exception = catchThrowable { client.posts() }
    // then
    verify(
        getRequestedFor(urlEqualTo("/posts"))
    )
    assertThat(exception).isInstanceOf(IllegalStateException::class.java)
}
```

### Database
- 개발 데이터와 테스트 데이터의 분리 필요 
- H2 database를 선택 후 프로파일에 test 설정
