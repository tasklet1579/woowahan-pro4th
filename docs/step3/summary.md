### ✏️ ATDD

요구사항을 검증하는 테스트로 소프트웨어를 개발하는 프로세스

TDD는 작은 단위의 요구 사항을 테스트하지만 ATDD는 시나리오 형태의 요구 사항을 테스트한다.

인수 테스트는 블랙 박스 테스트의 성격을 가지고 있기 때문에 클라이언트는 결과물의 내부 구현이나 사용된 기술을 기반으로 검증하기 보다는 표면적으로 확인할 수 있는 요소를 바탕으로 검증하려고 한다.

🧰 인수 테스트 도구

Spring Boot Test
- 테스트에 사용할 ApplicationContext를 쉽게 지정하게 도와줌
- 기존 @ContextConfiguration의 발전된 기능
- SpringApplication에서 사용하는 ApplicationContext를 생성해서 작동
- [Testing Spring Boot Applications](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing.spring-boot-applications)

WebEnvironment
- @SpringBootTest의 webEnvironment 속성을 사용하여 테스트 서버의 실행 방법을 설정
  - MOCK: Mocking된 웹 환경을 제공, MockMvc를 사용한 테스트를 진행할 수 있음
  - RANDOM_PORT: 실제 웹 환경을 구성
  - DEFINED_PORT: 실제 웹 환경을 구성, 지정한 포트를 listen
  - NONE: 아무런 웹 환경을 구성하지 않음

RestAssured
- REST-assured는 REST API의 테스트 및 검증을 단순화하도록 설계
- HTTP 작업에 대한 검증을 위한 풍부한 API를 활용 가능

```
@DisplayName("지하철역 관련 기능")
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.DEFINED_PORT)
public class StationAcceptanceTest {
    @LocalServerPort
    int port;
    
    @BeforeEach
    public void beforeEach() {
        if (RestAssured.port == RestAssured.UNDEFINED_PORT) {
            RestAssured.port = port;
        }
    }
    
    @Test
    void createStation() {
        ...
                
        createStation(request);
    }
    
    ExtractableResponse<Response> createStation(StationRequest request) {
        return RestAssured.given().log().all()
                          .body(request)
                          .contentType(MediaType.APPLICATION_JSON_VALUE)
                          .when().post("/stations")
                          .then().log().all()
                          .extract();
    }
}
```

🆚 MockMvc vs WebTestClient vs RestAssured
- MockMvc
  - @SpringBootTest의 webEnvironment.MOCK과 함께 사용 가능하며 mocking 된 web environment(ex tomcat) 환경에서 테스트
- WebTestClient
  - @SpringBootTest의 webEnvironment.RANDOM_PORT나 DEFINED_PORT와 함께 사용, Netty를 기본으로 사용
- RestAssured
  - 실제 web environment(Apache Tomcat)을 사용하여 테스트

🧪 테스트 격리

@DirtiesContext
- 효과적인 테스트 수행을 위해 스프링에서는 context caching 기능 지원
- @DirtiesContext를 활용하여 캐시 기능을 사용하지 않게 설정
- 매번 Context를 새로 구성하다보니 시간이 많이 걸림

DatabaseCleanup
- EntityManager를 활용하여 테이블 이름 조회 후 각 테이블 Truncate 수행
- ID auto increment 숫자를 1로 복구 시킴

[인수테스트에서 테스트 격리하기](https://tecoble.techcourse.co.kr/post/2020-09-15-test-isolation/)

### ✏️ @Transactional(readOnly = true)

```
A boolean flag that can be set to true if the transaction is effectively read-only, allowing for corresponding optimizations at runtime.  
Defaults to false.  
This just serves as a hint for the actual transaction subsystem; it will not necessarily cause failure of write access attempts. 
A transaction manager which cannot interpret the read-only hint will not throw an exception when asked for a read-only transaction but rather silently ignore the hint. 
```

- 의도지 않게 데이터를 변경하는 것을 막아준다.
- Hibernate를 사용하는 경우에는 FlushMode를 Manual로 변경하여 DIRTY CHECKING 생략이 가능하고 그로 인해 속도 향상 효과를 얻는다.
- 데이터베이스에 따라 DataSource Connection 레벨에도 설정되어 약간의 최적화가 가능하다.

```
package org.springframework.orm.jpa.vendor;

public class HibernateJpaDialect extends DefaultJpaDialect {
    ...
    
    @Override
	public Object beginTransaction(EntityManager entityManager, TransactionDefinition definition)
	    ...
	    
	    // Adapt flush mode and store previous isolation level, if any.
	    FlushMode previousFlushMode = prepareFlushMode(session, definition.isReadOnly());
	    
	    ...
	}
    
    @Nullable
    protected FlushMode prepareFlushMode(Session session, boolean readOnly) throws PersistenceException {
        FlushMode flushMode = session.getHibernateFlushMode();
        if (readOnly) {
            // We should suppress flushing for a read-only transaction.
            if (!flushMode.equals(FlushMode.MANUAL)) {
                session.setHibernateFlushMode(FlushMode.MANUAL);
                return flushMode;
            }
        }
        ...
    }
}
```

```
package org.hibernate.internal;

public class SessionImpl
        extends AbstractSessionImpl
        implements EventSource, SessionImplementor, HibernateEntityManagerImplementor {
		
    @Override
    public void flushBeforeTransactionCompletion() {
        final boolean doFlush = isTransactionFlushable() 
            && getHibernateFlushMode() != FlushMode.MANUAL;
        
        try {
            if ( doFlush ) {
              managedFlush();
            }
        }
        catch (RuntimeException re) {
            throw ExceptionMapperStandardImpl.INSTANCE.mapManagedFlushFailure( "error during managed flush", re, this );
        }
    }
```

위에서 readOnly이면 Hibernate Session의 Flush 모드를 MANUAL로 강제했고 doFlush는 false라서 managedFlush 메서드는 호출되지 않는다.

```
package org.springframework.data.jpa.repository.support;

@Repository
@Transactional(readOnly = true)
public class SimpleJpaRepository<T, ID> implements JpaRepositoryImplementation<T, ID> {
    private final EntityManager em;
    
    ...
    
    @Transactional
    @Override
    public void deleteById(ID id) {
        ...
    }
    
    @Override
    public Optional<T> findById(ID id) {
        ...
    }
    
    @Transactional
    @Override
    public <S extends T> S save(S entity) {
        ...
    }
}
```

인터페이스의 기본 구현체인 SimpleJpaRepository의 코드를 보면 readOnly 설정을 확인할 수 있다. 

### ✏️ @Embedded And @Embeddable

[출처](https://www.baeldung.com/jpa-embedded-embeddable)

### ✏️ Raw 타입은 사용하지 말자

[출처](https://mangkyu.tistory.com/137)

### ✏️ Map보다 DTO 클래스를 사용해야 하는 이유

[출처](https://mangkyu.tistory.com/164)

### ✏️ Jackson ObjectMapper

[jackson 라이브러리는 어떻게 동작하는가?](https://beaniejoy.tistory.com/75)

[ObjectMapper는 Property를 어떻게 찾을까?](https://bactoria.github.io/2019/08/16/ObjectMapper%EB%8A%94-Property%EB%A5%BC-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%B0%BE%EC%9D%84%EA%B9%8C/)

[@Request Body에서는 Setter가 필요없다?](https://jojoldu.tistory.com/407)
