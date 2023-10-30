# Elasti Cache란?
- Amazone ElasticCache는 인메모리 데이터베이스 캐싱 시스템을 제공한다.
- 애플리케이션이 데이터를 검색할 수 있는 성능, 속도 및 중복성 향상 서비스이다.
- Memcached와 Redis로 나누어진다.
  - 두개 다 이미 존재하나, AWS에서 사용 가능하도록 구현됨
  - Key - Value 값으로 이루어져 있다.
- RDS와 다르게 AWS 외부에서 접속할 수 <b>없다.</b>

### Redis란?
- Key, Value 구조의 비정형 데이터를 저장하고 관리하기 위한 오픈 소스 기반의 비관계형 데이터베이스 관리 시스템이다.
- 데이터베이스, 캐시, 메세지 브로커로 사용되며 인메모리 데이터 구조를 가진 저장소이다. 
- 사용이우
  - 데이터베이스는 데이터를 물리 디스크에 직접 쓰기 때문에 서버에 문제가 발생하여 다운되더라도 데이터가 손실되지 않는다. 하지만 매번 디스크에 접근해야 하기 때문에 사용자가 많아질수록 부하가 많아져서 느려질 수 있다.

### Spring Code

```java
@Configuration
@EnableConfigurationProperties(RedisProperties.class)
@RequiredArgsConstructor
public class RedisConfig {
    private final RedisProperties properties;
    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        LettuceConnectionFactory lettuceConnectionFactory = new LettuceConnectionFactory(properties.getHost(), properties.getPort());
        return lettuceConnectionFactory;
    }

    @Bean
    public RedisTemplate<String, Object> redisTemplate() {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory());
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        return redisTemplate;
    }
}

@Configuration
@EnableConfigurationProperties(RedisProperties.class)
@RequiredArgsConstructor
public class RedisConfig {
  private final RedisProperties properties;
  @Bean
  public RedisConnectionFactory redisConnectionFactory() {
    LettuceConnectionFactory lettuceConnectionFactory = new LettuceConnectionFactory(properties.getHost(), properties.getPort());
    return lettuceConnectionFactory;
  }

  @Bean
  public RedisTemplate<String, Object> redisTemplate() {
    RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
    redisTemplate.setConnectionFactory(redisConnectionFactory());
    redisTemplate.setKeySerializer(new StringRedisSerializer());
    return redisTemplate;
  }
}
```

### IAM을 통한 인증
참고 : https://docs.aws.amazon.com/ko_kr/AmazonElastiCache/latest/red-ug/auth-iam.html
- AWS IAM 자격 증명을 사용하여 레디스 연결을 인증할 수 있다
- 보안이 강화된다 보안을 위한 관리가 단순해진다.

