### ✏️ 자바의 리플렉션

```
import java.lang.reflect.Field;

public class Console {
    private static Scanner scanner;
    
    ...
    
    private static boolean isClosed() {
        try {
            final Field sourceClosedField = Scanner.class.getDeclaredField("sourceClosed");
            sourceClosedField.setAccessible(true);
            return sourceClosedField.getBoolean(scanner);
        } catch (final Exception e) {
            System.out.println("unable to determine if the scanner is closed.");
        }
        return true;
    }
}
```

자바의 리플렉션은 클래스 구조를 분석하여 동적 로딩을 할 수 있다.

new 키워드를 사용하지 않아도 인스턴스를 생성할 수 있고 프로그램 재실행없이 소스 코드를 교체할 수 있는 데다가 탐색을 통해서 외부에서 인스턴스를 할당할 수 있다.

단, 탐색을 하는데 상당한 비용이 들어가고 이미 메모리에 로드가 되었는데 소스 코드가 교체된다면 에러가 발생할 수 있다.

리플렉션의 활용적인 측면을 놓고 봤을 때 동적으로 할당된 클래스를 Object에 넣게 되면 Object의 기본 메서드뿐만 아니라 해당 클래스의 구조를 분석해서 메서드를 검색하고 실행할 수 있다.

getMethod를 사용하면 같은 이름의 메서드라도 파라미터의 설정에 따라서 호출을 다르게 할 수고 생성자 역시 다양하게 획득할 수 있는데 getMethod 이외에도 아래의 메서드를 사용할 수 있다.

- getMethods : Object에서부터 해당 클래스까지 있는 모든 메서드를 찾는다.
- getDeclaredMethods : 해당 클래스에 있는 메서드만을 찾는다.

이 기능을 잘 활용하면 private이나 protected처럼 접근이 제한되어 있는 메서드도 실행할 수 있기 때문에 테스트에서 메서드의 결과를 확인할 수 있다.

Console에서 Scanner는 정적으로 선언하였고 getDeclaredField로 sourceClosed 변수에 접근을 가능하게 하였는데 이렇게 프로그램 실행 도중 클래스의 멤버 변수에 접근해서 값을 확인하면 실제로 Scanner가 close 되었는지 알 수 있다.

### ✏️ 일급 컬렉션과 불변 객체

Collection을 Wrapping하면서, Wrapping한 Collection 외 다른 멤버 변수가 없는 상태를 일급 컬렉션이라고 한다.

```
public class Lottery {
    private final List<Number> numbers;
    
    public Lottery(List<Number> numbers) {
        validate(numbers);
        this.numbers = new LinkedList<>(numbers);
    }
    
    private void validate(List<Number> numbers) {
        ...
    }
    
    public List<Number> getNumbers() {
        return Collections.unmodifiableList(numbers);
    }
    
    public boolean contains(Number number) {
        return numbers.contains(number);
    }
    
    ...
}
```

일급 컬렉션의 장점
- 이름을 통해 의미를 부여할 수 있고 비즈니스에 종속적인 자료구조를 만들 수 있다.
- validation을 통해 객체의 상태를 보장할 수 있다.
- 로직을 한 곳에서 관리하기 때문에 유지보수에 유리하고 중복 코드를 줄일 수 있다.
- 필요에 따라서 객체의 불변성을 보장할 수 있고 목적에 맞는 값만 반환하는 것도 하나의 방법이 될 수 있다.

불변 객체
- 객체 생성 이후 내부의 상태가 변하지 않는 객체이다.
- read-only 메소드만을 제공하며, 방어적 복사를 통해 값을 제공한다.
- 대표적인 불변 객체로는 String이 있는데 내부적으로 문자열을 복사해서 반환한다.
- [불변 객체(Immutable Object) 및 final을 사용해야 하는 이유](https://mangkyu.tistory.com/131)

불변 객체 만들기
- 객체의 상태를 변경하는 메서드를 제공하지 않는다.
- 클래스를 확장할 수 없도록 한다. (public final class)
- 모든 필드를 final과 private으로 선언한다.
- 불변 객체의 인스턴스가 너무 많이 만들어진다면 정적 팩토리 메서드와 캐싱을 적용해 보자.

정적 팩토리 메서드
- 이름을 가짐으로써 객체 생성의 의도를 나타낼 수 있다.
- 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.
- 반환 타입의 하위 타입 객체를 반환할 수 있다.
- 입력 값에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

정적 팩토리 메서드 네이밍 규칙
- valudeOf(), of(), from() : 인자로 전달되는 값에 해당하는 인스턴스를 반환
- getInstance(), newInstance() : 보통 자신과 같은 Type의 인스턴스를 반환
- getType(), newType() : 자신과 다른 Type의 인스턴스를 반환

Q. [방어적 복사와 Unmodifiable의 차이점은?](https://steady-coding.tistory.com/559#%EB%B0%A9%EC%96%B4%EC%A0%81_%EB%B3%B5%EC%82%AC%EC%99%80_Unmodifiable%EC%9D%98_%EC%B0%A8%EC%9D%B4%EC%A0%90%EC%9D%80?)

> 방어적 복사는 A 리스트와 B 리스트 사이의 참조를 끊는 행위이지만, Unmodifiable은 참조를 끊지 않고 단순히 특정 리스트에서 요소의 변경이 일어날 경우 예외를 던진다. 그래서 생성자 단계에서 방어적 복사를 취하지 않고, getter에서 Unmodifiable만 취할 경우 초기 생성자로 주입한 컬렉션의 변화가 생기면 불변성이 깨진다.

Q. [방어적 복사를 사용하면 항상 불변성을 보장하는가?](https://steady-coding.tistory.com/559#%EB%B0%A9%EC%96%B4%EC%A0%81_%EB%B3%B5%EC%82%AC%EB%A5%BC_%EC%82%AC%EC%9A%A9%ED%95%98%EB%A9%B4_%ED%95%AD%EC%83%81_%EB%B6%88%EB%B3%80%EC%84%B1%EC%9D%84_%EB%B3%B4%EC%9E%A5%ED%95%98%EB%8A%94%EA%B0%80?)

> 그렇지 않다. 방어적 복사는 컬렉션의 요소에 대해 얕은 복사를 수행하므로 컬렉션의 참조 타입이 가변 객체라면, 복사하려는 컬렉션의 요소가 변경될 경우 불변성이 깨진다.

### ✏️ 객체지향적으로 개발하기

***Layered Architecture***
- Presentaion Layer : 사용자 화면을 구성
- Service Layer : 제어의 흐름을 관리
- Data Acess Layer : 데이터베이스와의 접속을 관리

***Service Layer와 Domain Object의 역할에 대한 오해***
- Service Layer에서 Business Logic을 개발하는 것은 절차지향적인 개발에 가깝다.
- Domain Object에서는 단순한 get, set등의 역할만 하고 있다.

```
// Service Layer
@Service
@Transactional
public class ContentService {    
    // 특정 회원이 작성한 게시물중 특정 시간 이후의 게시물만 가져온다.
    public List<Content> getContentsOfMemberAfterSpecificDate (Long memberId, DateTime date) {
        // 회원 조회
        Member member = memberRepository.getMember(memberId);
        
        List<Content> contents = member.getContents();
        List<Content> specificDateContents = Lists.newArrayList();
        
        // 특정 날짜 이후의 게시물만 가져오는 Business Logic
        for (Content content : contents) {
            // 특정 날짜 이후인지 체크하는 Business Logic
            DateTime contentCreatedAt = content.getCreatedAt();
            
            if (contentCreatedAt.isAfter(date)) {
                specificDateContents.add(content);
            }
        }

      return specificDateContents;
    }
}
```

특정 날짜 이후의 게시물을 가져오는 Logic과 해당 게시물이 특정 날짜 이후의 게시물인지 체크하는 Logic을 각각의 객체에게 위임해본다.

첫번째. 특정 날짜 이후의 게시물을 가져오는 Logic을 Member Object에 위임하고 해당 역할을 하는 메서드를 getContentsAfterSpecificDate로 명시한다.
```
// Service Layer
@Service
@Transactional
public class ContentService {
  @Autowired
  private MemberRepository memberRepository;

  @Autowired
  private ContentRepository contentRepository;

  ...

  public List<Content> getContentsOfMemberAfterSpecificDate (Long memberId, DateTime date) {
      Member member = memberRepository.getMember(memberId);

      // 특정 날짜 이후의 게시물을 가져오는 Business Logic을 Member 객체에 위임
      List<Content> specificDateContents = member.getContentsAfterSpecificDate(date);

      return specificDateContents;
  }

  ...

}
// Member Domain Object
@Entity
@JsonIgnoreProperties(ignoreUnknown = true)
public class Member {
    // 특정 날짜 이후의 게시물만 가져오는 Business Logic
    public List<Content> getContentsAfterSpecificDate (DateTime date) {
        List<Content> specificDateContents = Lists.newArrayList();
    
        for (Content content : contents) {
            DateTime contentCreatedAt = content.getCreatedAt();
            
            if (contentCreatedAt.isAfter(date)) {
                specificDateContents.add(content);
            }
        }
    }
}
```

두번째. 특정 날짜 이후의 데이터인지 체크하는 Logic을 Content에 위임하고 Member는 Content에 이 게시물이 특정 날짜 이후의 게시물인지 메시지를 보낸다.
```
// Member Domain Object
@Entity
@JsonIgnoreProperties(ignoreUnknown = true)
public class Member {
    public List<Content> getContentsAfterSpecificDate (DateTime date) {
        List<Content> specificDateContents = Lists.newArrayList();
        
        for (Content content : this.contents) {
            // Content 객체에게 특정날짜 이후인지 확인을 위한 메시지를 던짐
            if (content.isAfterCreatedDate(date)) {
                specificDateContents.add(content);
            }
        }
        
        return specificDateContents;
    }
}

// Content Domain Object
@Entity
public class Content {
    // 특정날짜 이후의 게시물인지 체크하는 Business Logic
    public Boolean isAfterCreatedDate (DateTime date) {
        return this.createdAt.isAfter(date);
    }
}
```

***Service Layer & Domain Object의 역할***

Service Layer
- 다른 기능의 Service를 호출하거나 다수의 DAO를 연결하는 역할을 한다.
- Transaction과 Cache 적용과 같은 infra 적용을 위한 단위가 된다.
- Service는 가능한 가볍게 구현한다.(thin layer)
- Service에 핵심 비즈니스 로직을 구현하지 말고, 로직은 상태 값을 가지고 있는 모델(또는 도메인)이 담당해야 한다.

Domain Object
- 객체는 상태값을 가지면서 행위 까지도 가진다.(State + Action)

[출처](https://wckhg89.tistory.com/13)

### ✏️ getter를 사용하는 대신 객체에 메시지를 보내자

객체는 캡슐화된 상태와 외부에 노출되어 있는 행동을 갖고 있으며, 다른 객체와 메시지를 주고 받으면서 협력한다. 객체는 메시지를 받으면 그에 따른 로직(행동)을 수행하게 되고, 필요하다면 객체 스스로 내부의 상태값도 변경한다. 간단히 말해서 객체지향 프로그래밍은 객체가 스스로 일을 하도록 하는 프로그래밍이다.

모든 멤버 변수에 getter를 생성해 놓고 상태값을 꺼내 그 값으로 객체 외부에서 로직을 수행한다면, 객체가 로직(행동)을 갖고 있는 형태가 아니고 메시지를 주고 받는 형태도 아니게 된다. 또한, 객체 스스로 상태값을 변경하는 것이 아니고, 외부에서 상태값을 변경할 수 있는 위험성도 생길 수 있다.

따라서 이는 객체가 객체스럽지 못한 것이다.

[출처](https://tecoble.techcourse.co.kr/post/2020-04-28-ask-instead-of-getter/)

### ✏️ 디미터 법칙

K. Lieberherr를 비롯한 몇몇 사람들은 디미터라는 이름의 프로젝트를 진행하던 도중 다른 객체들과의 협력을 통해 프로그램을 완성해나가는 객체지향 프로그래밍에서 객체들의 협력 경로를 제한하면 결합도를 효과적으로 낮출 수 있다는 사실을 발견했고 디미터 법칙을 만들었다.

현재 디미터 법칙은 객체 간 관계를 설정할 때 객체 간의 결합도를 효과적으로 낮출 수 있는 유용한 지침 중 하나로 꼽히며 객체 지향 생활 체조 원칙 중 한 줄에 점을 하나만 찍는다.로 요약되기도 한다.

[출처](https://tecoble.techcourse.co.kr/post/2020-06-02-law-of-demeter/)
