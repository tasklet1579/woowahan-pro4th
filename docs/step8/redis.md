✔️ Redis
```
package nextstep.subway.config;

import ...

@Profile("prod")
@EnableCaching
@Configuration
public class CacheConfig extends CachingConfigurerSupport {
    public static final long DEFAULT_EXPIRE_SECONDS = 60;
    public static final String STATION = "station";
    public static final String LINE = "line";
    public static final String PATH = "path";

    @Autowired
    RedisConnectionFactory connectionFactory;

    @Bean
    public CacheManager redisCacheManager() {
        RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig()
                                                                                 .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
                                                                                 .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));

        Map<String, RedisCacheConfiguration> cacheConfigurations = new HashMap<>();
        cacheConfigurations.put(CacheConfig.STATION, RedisCacheConfiguration.defaultCacheConfig()
                                                                            .entryTtl(Duration.ofHours(CacheConfig.DEFAULT_EXPIRE_SECONDS)));
        cacheConfigurations.put(CacheConfig.LINE, RedisCacheConfiguration.defaultCacheConfig()
                                                                         .entryTtl(Duration.ofHours(CacheConfig.DEFAULT_EXPIRE_SECONDS)));
        cacheConfigurations.put(CacheConfig.PATH, RedisCacheConfiguration.defaultCacheConfig()
                                                                         .entryTtl(Duration.ofHours(CacheConfig.DEFAULT_EXPIRE_SECONDS)));

        RedisCacheManager redisCacheManager = RedisCacheManager.RedisCacheManagerBuilder.fromConnectionFactory(connectionFactory)
                                                                                        .cacheDefaults(redisCacheConfiguration)
                                                                                        .withInitialCacheConfigurations(cacheConfigurations)
                                                                                        .build();
        return redisCacheManager;
    }
}
```

스레드풀을 필요 이상으로 크게 설정하면 DB 부하가 증가할 수 있다. 애플리케이션 서버가 낼 수 있는 최대 성능을 넘어서는 동시처리 요청이 들어오면 TPS가 증가하지 않은 채 응답시간만 증가하다가 큐가 쌓여 서비스 멈춤 현상이 발생할 수 있다.

애플리케이션 캐시를 활용하여 기존에 작업한 결과를 저장해두었다가 이후에 다시 동일한 작업이 수행되었을 때 결과를 재사용하면 반복되는 로직을 제거할 수 있고  병렬 처리 등을 활용하여 제한된 스레드 수 내에서 자원을 재사용하여 성능을 개선할 수 있다.

```
package nextstep.subway.line.application;

import ...

@Service
@Transactional
public class LineService {
    ...

    @Transactional(readOnly = true)
    @Cacheable(value = CacheConfig.LINE, unless = "#result == null or #result.isEmpty()")
    public List<Line> findLines() {
        return lineRepository.findAll();
    }

    @Caching(evict = {
            @CacheEvict(value = CacheConfig.LINE, allEntries = true),
            @CacheEvict(value = CacheConfig.PATH, allEntries = true)
    }
    )
    public void updateLine(Long id, LineRequest lineUpdateRequest) {
        Line persistLine = lineRepository.findById(id).orElseThrow(RuntimeException::new);
        persistLine.update(new Line(lineUpdateRequest.getName(), lineUpdateRequest.getColor()));
    }

    @Caching(evict = {
                @CacheEvict(value = CacheConfig.LINE, allEntries = true),
                @CacheEvict(value = CacheConfig.PATH, allEntries = true)
            }
    )
    public void deleteLineById(Long id) {
        lineRepository.deleteById(id);
    }
    
    ...
}
```

@Cacheable
- 메서드 실행 전에 캐시를 확인하여 최소 하나의 캐시가 존재한다면 값을 반환한다.

@CachePut
- 메서드 실행에 영향을 주지 않고 캐시를 갱신해야 하는 경우 사용한다.

@CacheEvict
- 캐시를 제거할 때 사용한다.
