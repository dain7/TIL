### Header에서 꺼내는 방법

```java
import javax.servlet.http.HttpServletRequest;

@RestController
public class MyController {

    @GetMapping("/api")
    public ResponseEntity<String> myApi(HttpServletRequest request) { // servlet을 통한 요청
        String ipAddress = request.getRemoteAddr(); // ip 주소를 꺼내옴
        // IP 주소를 사용하여 필요한 작업을 수행합니다.
        // ...
        return ResponseEntity.ok("API 호출이 완료되었습니다.");
    }
}
```

```java
@RestController
public class MyController {

    // 프록시 서버가 아닌 가장 첫번째 요청한 ip 조회 - X_FORWARDED_FOR
    @GetMapping("/api")
    public ResponseEntity<String> myApi(@RequestHeader(HttpHeaders.X_FORWARDED_FOR) String ipAddress) {
        if (ipAddress != null && ipAddress.contains(",")) {
            // IP 주소 목록에서 가장 첫 번째 IP 주소를 사용
            ipAddress = ipAddress.split(",")[0].trim();
        }

        // IP 주소를 사용하여 필요한 작업을 수행합니다.
        // ...

        return ResponseEntity.ok("API 호출이 완료되었습니다.");
    }
}
```

### Intercetor & thread local 이용

```kotlin
import org.springframework.stereotype.Component
import org.springframework.web.servlet.HandlerInterceptor
import javax.servlet.http.HttpServletRequest
import javax.servlet.http.HttpServletResponse

@Component
class IpLoggingInterceptor : HandlerInterceptor {

    override fun preHandle(request: HttpServletRequest, response: HttpServletResponse, handler: Any): Boolean {
        val userIp = getClientIp(request)
        UserIpHolder.setUserIp(userIp)
        // Continue with the execution chain
        return true
    }

    private fun getClientIp(request: HttpServletRequest): String {
        return request.getHeader("X-Forwarded-For") ?: request.remoteAddr
    }
}

import org.springframework.stereotype.Component

@Component
object UserIpHolder {

    private val userIpThreadLocal = ThreadLocal<String>()

    fun getUserIp(): String? {
        return userIpThreadLocal.get()
    }

    fun setUserIp(userIp: String) {
        userIpThreadLocal.set(userIp)
    }

    fun clearUserIp() {
        userIpThreadLocal.remove()
    }
}

import org.springframework.beans.factory.annotation.Autowired
import org.springframework.stereotype.Service

@Service
class MyService @Autowired constructor(private val userRepository: UserRepository) {

    fun someServiceMethod() {
        val userIp = UserIpHolder.getUserIp()
        // Use the user's IP address as needed in your service logic
        // ...
        UserIpHolder.clearUserIp() // Optional: Clear the ThreadLocal after usage
    }
}
```
