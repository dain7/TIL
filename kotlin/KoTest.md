# Kotest란?
- 테스트 코드 프레임워크

## JUnit의 단점 
- 한눈에 given, when, then 구분이 어렵다.
- 중복코드가 많다. 
- 테스트 스타일이 한정적이라 단위테스트에 적합하다.

## Kotest의 장점
- nested test code의 장점을 가져올 수 있다.
- DSL((Given(When(Then))))과 같은 구성으로 좀 더 명확하게 구분을 지을 수 있다.
- 다양한 test layout을 제공한다.

## 테스트 스타일

### Annotation Spec
- JUnit과 유사한 스타일
- Kotest -> JUnit으로 마이그레이션 할 때 좋음

```kotlin
import io.kotest.core.spec.style.AnnotationSpec

class UserTest : AnnotationSpec() {
    @Test
    fun `회원의 비밀번호와 일치하는지 확인한다()`{
        val user = createUser()
        shouldNotThrowAny { user.authenticate(PASSWORD) }
    }

    @Test
    fun `회원의 비밀번호와 다를 경우 예외가 발생한다`() {
        val user = createUser()
        shouldThrow<UnidentifiedUserException> { user.authenticate(WRONG_PASSWORD) }
    }
}
```

### String Spec
- @Test Annotation 없이 사용 가능하다.
- fun 없이 String으로 바로 테스트명 짓기 가능

```kotlin
import io.kotest.core.spec.style.StringSpec

class UserTest : StringSpec({
    "회원의 비밀번호와 일치하는지 확인한다" {
        val user = createUser()
        shouldNotThrowAny { user.authenticate(PASSWORD) }
    }

    "회원의 비밀번호와 다를 경우 예외가 발생한다" {
        val user = createUser()
        shouldThrow<UnidentifiedUserException> { user.authenticate(WRONG_PASSWORD) }
    }
})
```

### Behavior Spec
- Given, When, Then 형식을 사용하고 싶을 때 사용한다.

```kotlin
import io.kotest.core.spec.style.BehaviorSpec

internal class UserServiceTest : BehaviorSpec({
    val userRepository: UserRepository = mockk()
    val passwordGenerator: PasswordGenerator = mockk()
    
    Given("유저의 비밀번호가 주어질 때") {
        When("비밀번호를 변경하려 하면") {
            var request: EditPasswordRequest = mockk()

            slot<Long>().also { slot ->
                every { userRepository.getById(capture(slot)) } answers { createUser(id = slot.captured) }
            }

            Then("확인용 비밀번호가 일치한다면 변경한다") {
                // ...
            }

            Then("확인용 비밀번호가 일치하지 않으면 예외가 발생한다") {
                // ...
            }
        }
    }
})
```