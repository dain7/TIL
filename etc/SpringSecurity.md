# Spring Security

### Security Config

```kotlin
@Configuration
@EnableWebSecurity
class SecurityConfiguration(
    private val jwtAuthProvider: JwtAuthProvider,
    private val jwtAuthenticationEntryPoint: JwtAuthDeniedHandler,
    private val jwtAccessDeniedHandler: JwtAccessDeniedHandler
) {
    @Bean
    fun filterChain(httpSecurity: HttpSecurity): SecurityFilterChain {
        httpSecurity
            .csrf { it.disable() }
            .exceptionHandling {
                it.authenticationEntryPoint(jwtAuthenticationEntryPoint) // 인증 실패 진입점
                it.accessDeniedHandler(jwtAccessDeniedHandler) // 인가 실패 진입점
            }
            .authorizeHttpRequests {  // HttpServletRequest를 사용하는 요청들에 대한 접근제한을 설정하겠다.
                it.requestMatchers("/hello", "/auth/**", "/virtual-accounts/**", "/wallets/**").permitAll()
                it.anyRequest().authenticated() // 그 외 인증없이 접근 불가
            }
            .addFilterBefore(JwtAuthFilter(jwtAuthProvider), UsernamePasswordAuthenticationFilter::class.java)
            .sessionManagement { it.sessionCreationPolicy(SessionCreationPolicy.STATELESS) }
        return httpSecurity.build()
    }
}
```

### JwtAuthProvider

JWT를 생성하고 검증하는 컴포넌트

```kotlin
@Service
class JwtAuthProvider(@Value("\${jwt.secret}") private val secret: String) {
    private val key = hmacShaKeyFor(secret.toByteArray())

    fun create(merchantId: String): String {
        val claims = generateClaims(merchantId) // 클레임 생성

        val expiration = Date(System.currentTimeMillis() + 10 * 60 * 1000) // 발급일
        return Jwts.builder()
            .setHeaderParam("typ", "JWT") // 헤더
            .setClaims(claims) // 클레임 담기
            .setExpiration(expiration) // 만료시간
            .setIssuedAt(Date()) // 발급일
            .signWith(key, SignatureAlgorithm.HS256)
            .compact()
    }

    private fun generateClaims(merchantId: String): Claims {
        return Jwts.claims().apply {
            put("mId", merchantId)
            put("auth", "OWNER") // 임시로 모든 권한 ONWER로 넣어놨습니다. 권한체크는 추후에 작성 부탁드립니다. 아마 권한도 코어에서 넣어줘야 할듯 합니다.
        }
    }

    // Request Header 에서 토큰 정보를 꺼내오기 위한 메소드
    fun resolveToken(request: HttpServletRequest): String? {
        val bearerToken = request.getHeader("Authorization")

        if (StringUtils.hasText(bearerToken) && bearerToken.startsWith("Bearer ")) {
            return bearerToken.substring(7)
        }

        return null
    }

    // 실제로 우리가 검증할 함수
    fun validateToken(token: String): Boolean {
        return try {
            Jwts.parserBuilder()
                .setSigningKey(key)
                .build()
                .parseClaimsJws(token)
            true
        } catch (e: Exception) {
            false
        }
    }

    // 권한과 mid를 가져온다
    fun getAuthentication(token: String): Authentication {
        // 클레임 정보 읽기
        val claims: Claims = Jwts.parserBuilder()
            .setSigningKey(key)
            .build()
            .parseClaimsJws(token)
            .body

        val authorities: Collection<GrantedAuthority> =
            claims["auth"].toString().split(",")
                .map { SimpleGrantedAuthority(it) }

        val principal = User(claims["mId"].toString(), "", authorities)

        return UsernamePasswordAuthenticationToken(principal, token, authorities)
    }
}
```

### AuthenticationEntryPoint

유효한 자격증명을 제공하지 않고 접근하려 할때 401 Unauthorized 에러를 리턴하는 class

```kotlin
@Component
class JwtAuthDeniedHandler : AuthenticationEntryPoint {

    override fun commence(
        request: HttpServletRequest,
        response: HttpServletResponse,
        authException: AuthenticationException
    ) {
        val responseBody = generateError<Unit>().toJsonString()
        response.contentType = MediaType.APPLICATION_JSON_VALUE
        response.status = HttpStatus.UNAUTHORIZED.value()
        response.characterEncoding = "UTF-8"
        response.writer.write(responseBody)
    }

    private fun <T> generateError(): ApiResponse<T> {
        return ApiResponse(
            status = ResponseStatus(
                code = ResultCode.AUTHENTICATION_FAILED.toString(),
                message = ResultCode.AUTHENTICATION_FAILED.message,
                traceId = UUID.randomUUID().toString()
            )
        )
    }
}
```

### JwtAccessDeniedHandler

필요한 권한이 존재하지 않는 경우에 403 Forbidden 에러를 리턴하는 class

```kotlin
@Component
class JwtAccessDeniedHandler : AccessDeniedHandler {
    override fun handle(
        request: HttpServletRequest?,
        response: HttpServletResponse?,
        accessDeniedException: org.springframework.security.access.AccessDeniedException?
    ) {
        response?.sendError(HttpServletResponse.SC_FORBIDDEN)
    }
}
```

### JwtFilter

실제로 이 컴포넌트를 이용하는 것은 인증 작업을 진행하는 Filter
이 필터는 검증이 끝난 JWT로부터 유저정보를 받아와서 UsernamePasswordAuthenticationFilter로 전달

```kotlin
@Component
class JwtAuthFilter(private val jwtAuthProvider: JwtAuthProvider) : GenericFilterBean() {
    override fun doFilter(
        request: ServletRequest,
        response: ServletResponse,
        chain: FilterChain
    ) {
        // 토큰의 인증 정보를 SecurityContext에 저장하는 역할 수행
        val httpRequest = request as HttpServletRequest
        val jwt = jwtAuthProvider.resolveToken(httpRequest)

        if (jwt?.isNotBlank() == true && jwtAuthProvider.validateToken(jwt)) {
            val authentication = jwtAuthProvider.getAuthentication(jwt)
            SecurityContextHolder.getContext().authentication = authentication
        }

        chain.doFilter(request, response)
    }
}
```
