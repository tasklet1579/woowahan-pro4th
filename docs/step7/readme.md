### 레거시 코드 리팩터링

🎯 학습 목표

많은 기업들이 "서비스를 안정적으로 운영하면서 레거시 코드를 리팩터링할 수 있는 역량을 갖춘 개발자"를 요구한다.
- 레거시 프로젝트를 리팩터링하는 경험을 통해 서비스를 안정적으로 운영하면서 레거시 코드를 리팩터링할 수 있는 역량을 키운다.
- 프로젝트를 만드는 단계에서 끝나는 것이 아니라 프로젝트를 완료한 후 일정 기간 유지보수를 함으로써 레거시 코드를 리팩터링하는 경험을 쌓는다.

주요 내용
- 리팩터링 프로세스와 일반 원칙 이해하기
- 프로그램을 더 쉽게 이해하고 변경하는 유용한 리팩터링 빠르게 적용하기
- 리팩터링 가능성이 있는 코드에서 풍기는 악취 인식하기
- 각 리팩터링 기법의 개념, 동기부여, 역학 및 간단한 사례 살펴보기
- 리팩터링을 수행하는 견고한 테스트 구축하기
- 리팩터링의 장단점과 장애물 인식하기

---

### 1. 테스트를 통한 코드 보호

✏️ 요구 사항
- kitchenpos 패키지의 코드를 보고 키친포스의 요구 사항을 [마크다운](https://dooray.com/htmls/guides/markdown_ko_KR.html) 문법을 참고해 작성한다.
- 정리한 키친포스의 요구 사항을 토대로 테스트 코드를 작성한다. 모든 Business Object에 대한 테스트 코드를 작성한다.
- @SpringBootTest를 이용한 통합 테스트 코드 또는 @ExtendWith(MockitoExtension.class)를 이용한 단위 테스트 코드를 작성한다.
  - [Testing in Spring Boot](https://www.baeldung.com/spring-boot-testing)
  - [Exploring the Spring Boot TestRestTemplate](https://www.baeldung.com/spring-boot-testresttemplate)

```
###
POST {{host}}/api/menu-groups
Content-Type: kitchenpos.application/json

{
  "name": "추천메뉴"
}

###
GET {{host}}/api/menus-groups

###

```

src/main/resources/db/migration 디렉터리의 .sql 파일을 보고 어떤 관계로 이루어져 있는지 참고한다.
```
id BIGINT(20) NOT NULL AUTO_INCREMENT,
order_table_id BIGINT(20) NOT NULL,
order_status VARCHAR(255) NOT NULL,
ordered_time DATETIME NOT NULL,
PRIMARY KEY (id)
```

아래의 예제를 참고한다.
```
### 상품

* 상품을 등록할 수 있다.
* 상품의 가격이 올바르지 않으면 등록할 수 없다.
    * 상품의 가격은 0 원 이상이어야 한다.
* 상품의 목록을 조회할 수 있다.
```

---

### 2. 서비스 리팩터링

✏️ 요구 사항

단위 테스트하기 어려운 코드와 단위 테스트 가능한 코드를 분리해 단위 테스트 가능한 코드에 대해 단위 테스트를 구현한다.
- Spring Data JPA 사용 시 spring.jpa.hibernate.ddl-auto=validate 옵션을 필수로 준다.
- 데이터베이스 스키마 변경 및 마이그레이션이 필요하다면 아래 문서를 적극 활용한다.
  - [DB도 형상관리를 해보자!](https://meetup.toast.com/posts/173)

비즈니스 로직은 어느 곳에 구현하는 것이 좋을까?

다음과 같은 계층형 아키텍처 기반 하에서 핵심 비즈니스 로직은 어디에 구현하는 것이 맞을까?

![image](../image/step7/image01.png)

응용 애플리케이션을 개발할 때 TDD, OOP를 적용하려면 핵심 비즈니스 로직을 도메인 객체가 담당하도록 구현하는 것이다.
즉, 테스트하기 쉬운 부분과 테스트하기 어려운 부분을 분리해 테스트하기 쉬운 부분에 대한 단위 테스트를 구현하고 지속적인 리팩터링을 한다.

---

### 3. 의존성 리팩터링

✏️ 요구 사항

이전 단계에서 객체 지향 설계를 의식하였다면 아래의 문제가 존재한다. 의존성 관점에서 설계를 검토해 본다.
- 메뉴의 이름과 가격이 변경되면 주문 항목도 함께 변경된다. 메뉴 정보가 변경되더라도 주문 항목이 변경되지 않게 구현한다.
- 클래스 간의 방향도 중요하고 패키지 간의 방향도 중요하다. 클래스 사이, 패키지 사이의 의존 관계는 단방향이 되도록 해야 한다.
- 데이터베이스 스키마 변경 및 마이그레이션이 필요하다면 아래 문서를 적극 활용한다.
  - [DB도 형상관리를 해보자!](https://meetup.toast.com/posts/173)

힌트
- 함께 생성되고 함께 삭제되는 객체들을 함께 묶어라
- 불변식을 지켜야 하는 객체들을 함께 묶어라
- 가능하면 분리하라

연관 관계는 다양하게 구현할 수 있다.
- 직접 참조 (객체 참조를 이용한 연관 관계)
- 간접 참조 (리포지토리를 통한 탐색)

---

### 4. 멀티 모듈 적용

✏️ 요구 사항

- Gradle의 멀티 모듈 개념을 적용해 자유롭게 서로 다른 프로젝트로 분리해 본다.
  - 컨텍스트 간의 독립된 모듈로 만들 수 있다.
  - 계층 간의 독립된 모듈로 만들 수 있다.
- 의존성 주입, HTTP 요청/응답, 이벤트 발행/구독 등 다양한 방식으로 모듈 간 데이터를 주고받을 수 있다.

힌트

- [Gradle 멀티 프로젝트 관리](https://jojoldu.tistory.com/123)
- [Gradle Multi Project](https://kwonnam.pe.kr/wiki/gradle/multiproject)
