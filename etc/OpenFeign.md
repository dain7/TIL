# OpenFeign
- Feign Client는 넷플릭스에서 개발한 Http Client 이다.
- 인터페이스를 작성하고 어노테이션을 통해 따로 구현체 없이 Http Client를 구현할 수 있다.
- 여기서 Http Client란 간단히 얘기해서 특정 프로그램에서 Http 요청을 간편하게 만들어서 보낼 수 있도록 돕는 객체이다.

## 왜 Feign Client를 사용할까?
- Spring 프레임 워크로 웹 서버 프로그램을 설계하는데 있어 Http Client에 대해 다양한 후보군이 존재한다.

### 1. Rest template
- 스프링 프레임워크 이전 버전에서 주로 사용되었으며, 싑게 접근하고 사용 가능
- 동기식 호출 방식이기 때문에 단순 요청 처리에 효과적
- 대량의 요청처리나 비동기 처리가 필요한 경우에는 성능 이슈 발생 가능성

### 2. WebClient
- 스프링 프레임워크 5부터 도입된 비동기식 HTTP 클라이언트이다.
- 비동기 및 리액티브 스트림 처리를 지원하여 확장성과 성능면에서 우수
- 리액티브 스트림을 활용하여 대량의 요청을 효과적으로 처리할 수 있음
- 비동기 처리의 개념과 리액티브 프로그래밍에 익숙해져야 함.

### 3. OpenFeign
- 선언적인 REST 클라이언트로서 서비스 간의 통신을 자동화
- 인터페이스 기반의 API 정의로 간단하고 직관적인 코드 작성 가능
- 스프링 클라우드와의 통합을 지원하여 마이크로 서비스 아키텍처에서 유용
- 자동화 된 요청 처리와 부가적인 기능 지원으로 개발 생상선을 향상 

-> Rest Template은 Deprecated 되었고, Webclient는 비동기와 리액티브에 대한 러닝커브가 존재한다. Feign Client는 사용 방법 및 커스텀이 간편함


## 사용 방법
```java
@FeignClient(value = "example", url = "$")
public interface ExampleClient {

    @GetMapping("/status/")
    void status(@PathVariable("status") int status);
}
```
- @FeignClient 어노테이션 설정
  - name : feign client 이름
  - url : 호출할 api url
  - configuration : feign client 설정 정보가 세팅되어 있는 클래스


### Fallback
- Spring Cloud CircuitBreaker supports the notion of a fallback: 
- a default code path that is executed when the circuit is open or there is an error. 
- To enable fallbacks for a given @FeignClient set the fallback attribute to the class name that implements the fallback. 
- You also need to declare your implementation as a Spring bean.
- 
```java
@FeignClient(name = "test", url = "http://localhost:${server.port}/", fallback = Fallback.class)
    protected interface TestClient {

        @RequestMapping(method = RequestMethod.GET, value = "/hello")
        Hello getHello();

        @RequestMapping(method = RequestMethod.GET, value = "/hellonotfound")
        String getException();

    }

    @Component
    static class Fallback implements TestClient {

        @Override
        public Hello getHello() {
            throw new NoFallbackAvailableException("Boom!", new RuntimeException());
        }

        @Override
        public String getException() {
            return "Fixed response";
        }

    }
```

### Test
##### wiremock test
```kotlin
@ActiveProfiles({"test"})
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
@DisplayName("북마크 서비스레이어 통합 테스트")
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.NONE)
public class BookmarkServiceMockTest extends AbstractContainerTestBase {

    @Autowired
    private BookmarkCommandService bookmarkCommandService;

    static final int PORT = 8080;
    public static WireMockServer wireMockServer = new WireMockServer(options().port(PORT));

    @Autowired
    private PersistenceHelper persistenceHelper;

    @DynamicPropertySource
    public static void addUrlProperties(DynamicPropertyRegistry registry) {
        registry.add("api.authentication-server.url", () -> "localhost:" + PORT);
    }

    @BeforeAll
    public static void beforeAll() {
        wireMockServer.start();
        WireMock.configureFor("localhost", PORT);
    }

    @AfterAll
    public static void afterAll() {
        wireMockServer.stop();
    }

    @AfterEach
    public void afterEach() {
        wireMockServer.resetAll();
    }

    @Test
    @DisplayName("북마크 저장을 하면 북마크가 생긴다.")
    public void given_memberid_newsid_when_save_bookmark_then_is_not_null() {
        Long memberId = 1L;
        wireMockServer.stubFor(get(urlMatching("/api/member/exist/"+memberId))
                .willReturn(aResponse()
                        .withHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
                        .withBody("true"))
        );

        News persist = persistenceHelper.persist(NewsFixture.createNews());
        Bookmark bookmark = bookmarkCommandService.saveBookmark(1L, memberId);

        assertThat(bookmark).isNotNull();
    }

    @Test
    @DisplayName("Auth서버랑 연결되지 않으면 에러를 반환한다.")
    public void when_bookmark_save_then_throw_error() {
        assertThatThrownBy(() -> bookmarkCommandService.saveBookmark(1L, 1L))
                .isInstanceOf(IllegalArgumentException.class)
                .hasMessage("페인 클라이언트 에러!");
    }
}
```


# 참고자료
우아한 형제들 : 우아한 feign 적용기
https://techblog.woowahan.com/2630/