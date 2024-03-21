# IP Whitelist in spring
### 필요한 기능
- 요청 IP를 가져오는 Util
- IP 목록
- 요청 IP가 해당 리스트 내 존재하는지 체크
- 사용자의 요청을 가로채서 접근 가능한 IP인지 확인 (Interceptor)

### 예제코드
##### List와 IP 매칭 작업
  - IpAddressMatcher : Spring Security에서 제공하는 IP 매칭 여부 판단 객체 
```java
@Service
@RequiredArgsConstructor
public class AllowCidrCheckServiceImpl implements AllowCidrCheckService {

    private final WhiteCidrDAO whiteCidrDAO;

    /**
     * client IP를 받아서
     * 등록된 white CIDR 리스트에 포함되는지 체크
     * 
     * @param clientIp
     * @return boolean  - white IP (True), NOT white IP (False)
     */
    public boolean isWhiteIp(String clientIp) {

        return whiteCidrDAO.selectWhiteCidrList()
            .stream()
            .anyMatch(cidr ->
                (new IpAddressMatcher(cidr)).matches(clientIp)
            );
    }
}
```

##### Interceptor
```java
@Slf4j
@Component
@RequiredArgsConstructor
public class IpAccessControlInterceptor implements HandlerInterceptor {

    private final AllowCidrCheckService allowCidrCheckService;

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws IOException {
        String requestIp = getClientIp(request);

        if (!allowCidrCheckService.isWhiteIp(requestIp)) {
            String requestURI = request.getRequestURI();

            log.warn("Forbidden access. request uri={}, client ip={}", requestURI, requestIp);
            
            // redirect NOT AUTH PAGE or FORBIDDEN status
            response.sendError(403, "IP Forbidden");
            return false;
        }

        return true;
    }
}
```

##### Configuration
- Configuration에 interceptor 추가
```java
@Configuration
@RequiredArgsConstructor
public class WebMvcConfig implements WebMvcConfigurer {

    private final IpAccessControlInterceptor ipAccessControlInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {

        registry.addInterceptor(ipAccessControlInterceptor)
            .addPathPatterns("/**")
            .excludePathPatterns("/resources/**")
            .excludePathPatterns("/error/**");
    }
}
```

##### aha point
- 빈을 두번 등록하면 안됨 

```kotlin
@Configuration
class WebMvcConfiguration(
    private val whitelistConfig: WhitelistConfig
) : WebMvcConfigurer {

    override fun addInterceptors(registry: InterceptorRegistry) {
        registry.addInterceptor(baseHandlerInterceptor()).addPathPatterns("/**")
        registry.addInterceptor(ipAccessInterceptor()).addPathPatterns("/**")
    }

    @Bean
    fun baseHandlerInterceptor(): BaseHandlerInterceptor {
        return BaseHandlerInterceptor()
    }

    @Bean
    fun ipAccessInterceptor(): IpAccessInterceptor {
        return IpAccessInterceptor(whitelistConfig)
    }
}

```

```kotlin
// @Component 어노테이션 제거
open class IpAccessInterceptor(
    private val whitelistConfig: WhitelistConfig
) : HandlerInterceptor {

    private val log = logger()

    override fun preHandle(request: HttpServletRequest, response: HttpServletResponse, handler: Any): Boolean {
        val clientIp = getIpAddress()
        if(!isWhiteIp(clientIp)) {
            val requestUri = request.requestURI
            log.warn("Forbidden IP. request uri = {$requestUri}, client ip = {$clientIp}")

            response.sendError(403, "IP Forbidden")
            return false
        }

        return true
    }

    private fun isWhiteIp(clientIp: String): Boolean {
        return whitelistConfig.cidrs.any { cidr ->
            IpAddressMatcher(cidr).matches(clientIp)
        }
    }
}
```

- 얘는 테스트
```kotlin
@SpringBootTest
@AutoConfigureMockMvc
class IpAccessInterceptorTest {
    @Autowired
    lateinit var mockMvc: MockMvc

    @Test
    fun `Allowed IP`() {
        mockMvc.perform(get("/hello").header("X-Forwarded-For", "127.0.0.1"))
            .andExpect(status().isOk)
    }

    @Test
    fun `Not allowed IP`() {
        mockMvc.perform(get("/hello").header("X-Forwarded-For", "192.99.99.99"))
            .andExpect(status().isForbidden)
    }
}

```