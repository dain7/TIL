# path와 method를 받아 header에 담긴 중복체크 아이디 확인 인터셉터 구현 기록

### PathMatcherInterceptor
```kotlin
class PathMatcherInterceptor(
    private var handlerInterceptor: HandlerInterceptor
) : HandlerInterceptor {

    private var pathContainer: PathContainer = PathContainer()

    val log = logger()

    override fun preHandle(request: HttpServletRequest, response: HttpServletResponse, handler: Any): Boolean {
        log.info("request path: ${request.servletPath}, request method: ${request.method}")

        // 정의한 url에 포함되어 있지 않으면 서비스 로직으로 요청
        if (!pathContainer.isIncludedPath(request.servletPath, request.method)) return true

        // 아니라면 중복 거래 체크 인터셉트 진행
        return handlerInterceptor.preHandle(request, response, handler)
    }

    fun includePathPattern(path: String, method: HttpMethod) {
        pathContainer.addIncludePathPattern(path, method.toString())
    }

    fun excludePathPattern(path: String, method: HttpMethod) {
        pathContainer.addExcludePathPattern(path, method.toString())
    }
}

class PathContainer {
    private var includePathPattern: List<MultiValueMap<String, String>> = mutableListOf()
    private var excludePathPattern: List<MultiValueMap<String, String>> = mutableListOf()
    val log = logger()
    fun isIncludedPath(requestedPath: String, requestedMethod: String): Boolean {
        return includePathPattern.any { possiblePath -> isMatchPathPattern(requestedPath, withoutBrackets(requestedMethod), possiblePath)}
    }

    private fun isMatchPathPattern(requestedPath: String, requestedMethod: String, possiblePath: MultiValueMap<String, String>): Boolean {
        if (possiblePath[requestedPath].isNullOrEmpty()) return false
        return possiblePath[requestedPath]!!.contains(requestedMethod)
    }

    fun addIncludePathPattern(requestedPath: String, requestedMethod: String) {
        this.includePathPattern += addMap(requestedPath, requestedMethod)
    }

    fun addExcludePathPattern(requestedPath: String, requestedMethod: String) {
        this.excludePathPattern += addMap(requestedPath, requestedMethod)
    }

    private fun addMap(requestedPath: String, requestedMethod: String): MultiValueMap<String, String> {
        val map: MultiValueMap<String, String> = LinkedMultiValueMap()
        map.add(requestedPath, requestedMethod)
        return map
    }

    private fun withoutBrackets(str: String): String {
        return str.replace(Regex("[\\[\\]]"), "")
    }
}

```

### DuplicateTransactionInterceptor
```kotlin
open class DuplicateTransactionInterceptor(
    private var redisUtil: RedisUtil
) : HandlerInterceptor {

    private val log = logger()

    override fun preHandle(request: HttpServletRequest, response: HttpServletResponse, handler: Any): Boolean {
        val requestId = request.getHeader("request-id") ?: return true
        val merchantId = getMerchantId()
        val key = "${requestId}${merchantId}"
        log.info("request id: $requestId, merchant id: $merchantId key: $key")

        val result = redisUtil.getString(key)
        if (!result.isNullOrBlank()) {
            log.info("$key is already exists.")
            return false
        }
        redisUtil.setStringWithExpireSeconds(key, "value", 24*60*60)
        return true
    }
}
```