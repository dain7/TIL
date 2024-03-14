# Graphql
- api를 위한 쿼리 언어

### Rest API와의 차이
- GraphQL은 보통 하나의 엔드포인트를 가진다.
- GraphQL은 요청할 때 사용하는 쿼리에 따라 다른 응답을 받을 수 있다.
- GraphQL은 원하는 데이터(response)만 받을 수 있다.

### example
```json
// GraphQL request
query {
    person(personID: 1) {
        name
        height
        mass
    }
}

// GraphQL response
{
    "data": {
        "person": {
            "name": "Luke Skywalker",
            "height": 172,
            "mass": 77
        }
    }
}
```

### 장단점
##### 장점 
- http 요청 횟수를 줄일 수 있다.

##### 단점
- 캐싱이 rest보다 어렵다.

### 예시
```java
@Controller
public class TestController {

    private String testAString = "testAString";

    @QueryMapping
    public String testA() {
        return testAString;
    }

    @MutationMapping
    public String testAUpdate(@Argument String changeValue) {
        testAString = changeValue;
        return testAString;
    }
}
```