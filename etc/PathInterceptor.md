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
        val foundPattern = findPathPattern(requestedPath, possiblePath)
        if (possiblePath[requestedPath].isNullOrEmpty()) return false
        return possiblePath[requestedPath]!!.contains(requestedMethod)
    }

    private fun findPathPattern(requestedPath: String, possiblePath: MultiValueMap<String, String>): String? {
        return possiblePath.keys.firstOrNull { Regex(toNormalize(it)).matches(requestedPath) }
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

### aop로 분리

```kotlin
package com.sentbe.bizplatform.userapi.common.aop

import com.sentbe.bizplatform.userapi.exception.BadRequestException
import com.sentbe.bizplatform.userapi.exception.ResultCode
import com.sentbe.bizplatform.userapi.exception.SystemException
import com.sentbe.bizplatform.userapi.util.RedisUtil
import com.sentbe.bizplatform.userapi.util.getMerchantId
import com.sentbe.bizplatform.userapi.util.logger
import org.aspectj.lang.annotation.Aspect
import org.aspectj.lang.annotation.Before
import org.springframework.beans.factory.annotation.Value
import org.springframework.stereotype.Component
import org.springframework.web.context.request.RequestContextHolder
import org.springframework.web.context.request.ServletRequestAttributes

@Aspect
@Component
class DuplicateCheckAspect(
    @Value("\${duplicate-check.expiration-seconds}") var seconds: Long = 10,
    private var redisUtil: RedisUtil,
    private var includePathPattern: List<String> = listOf("/v1/wallets/*/convert")
) {

    val log = logger()

    @Before("@annotation(com.sentbe.bizplatform.userapi.common.aop.DupCheck)")
    fun duplicateCheck() {
        val request = (RequestContextHolder.currentRequestAttributes() as ServletRequestAttributes).request
        val duplicateCheckKey = request.getHeader("duplicate-check-key") ?: throw BadRequestException(ResultCode.PARAMETER_MISSED, "Duplicate Check Key is required.")
        val merchantId = getMerchantId()
        val path = getNormalizePattern(request.servletPath) ?: request.servletPath
        val key = "duplicateCheck:$path:${duplicateCheckKey}:${merchantId}"
        log.info("request path: $path, duplicate check key: $duplicateCheckKey, merchant id: $merchantId, key: $key")

        val result = redisUtil.getString(key)
        if (!result.isNullOrBlank() && result == duplicateCheckKey) {
            log.info("$duplicateCheckKey is already exists.")
            throw SystemException(ResultCode.INTERNAL_SERVER_ERROR, "Duplicated Request.")
        }
        redisUtil.setStringWithExpireSeconds(key, duplicateCheckKey, seconds)
    }

    private fun getNormalizePattern(requestedPath: String): String? {
        return includePathPattern.firstNotNullOfOrNull { possiblePath -> findPathPattern(requestedPath, possiblePath) }
    }

    private fun findPathPattern(requestedPath: String, possiblePath: String): String? {
        return if (Regex(toNormalize(possiblePath)).matches(requestedPath)) possiblePath else null
    }

    private fun toNormalize(str: String): String {
        return str.replace("*", ".*")
    }
}

```