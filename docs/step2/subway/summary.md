### ✏️ 지연 로딩

하이버네이트에서 지연 로딩은 프록시를 기반으로 한다. 

지연 로딩의 대상이 되는 객체는 곧바로 DB에 접근하여 값을 조회하지 않고 프록시 객체를 생성해서 영속성 컨텍스트에 저장하는데 실제로 해당 객체가 사용될 때 DB 조회 후 엔티리를 생성한다. 
이 때 실제 사용될 엔티티를 생성하는 행위를 영속성 초기화라고 한다.

***왜 지연 로딩을 사용할까?***

```
package com.shop.entity;

import ...

@Entity
public class Order extends BaseEntity {
    ...

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id")
    private Member member;

    @OneToMany(mappedBy = "order", cascade = ..., orphanRemoval = true, fetch = FetchType.LAZY)
    private List<OrderItem> orderItems = new ArrayList<>();
}
```

```
package com.shop.entity;

import ...

@SpringBootTest
class OrderTest {
    ...
    
    @Test
    public void lazyLoading(){
        ...
        Long orderItemId = order.getOrderItems().get(0).getId();
        em.flush();
        em.clear();
        
        OrderItem orderItem = orderItemRepository.findById(orderItemId)
                                                 .orElseThrow(EntityNotFoundException::new);
        ...
    }
}
```

주문 상품 엔티티를 조회하면 order_item 테이블과 item, orders, member 테이블을 조인해서 한꺼번에 데이터를 가져오는데 (@ManyToOne은 즉시 로딩이 기본 전략) 연관 관계에 따라서 현재의 비즈니스 로직에서 사용하지 않을 데이터도 가져오게 된다.

실제 비즈니스를 하고 있다면 매핑되는 엔티티의 개수는 훨씬 많을테고 그렇게 되면 쿼리가 어떻게 실행될지 예측할 수 없다. 또한 사용하지 않는 데이터도 함께 조회하므로 성능 문제도 있을 수 있다.

따라서 즉시 로딩을 사용하는 대신에 지연 로딩 방식을 사용해야 하고 필요한 경우에만 즉시 로딩으로 설정하면 된다.

### ✏️ CASCADE

영속성 전이는 엔티티의 상태를 변경할 때 해당 엔티티와 연관된 엔티티의 상태 변화를 전파시키는 옵션이다.

이 때 부모는 One에 해당하고 자식은 Many에 해당하는데 주문 엔티티가 삭제되었을 때 해당 엔티티와 연간되어 있는 주문 상품 엔티티가 함께 삭제되거나 주문 엔티티를 저장할 때 주문 상품 엔티티를 함께 저장할 수 있다.

|종류|설명|
|---|---|
|PERSIST|부모 엔티티가 영속화될 때 자식 엔티티도 영속화|
|MERGE|부모 엔티티가 병합될 때 자식 엔티티도 병합|
|REMOVE|부모 엔티티가 삭제될 때 연관된 자식 엔티티도 삭제|
|REFRESH|부모 엔티티가 refresh되면 연관된 자식 엔티티도 refresh|
|DETACH|부모 엔티티가 detach 되면 연관된 자식 엔티티도 detach 상태로 변경|
|ALL|부모 엔티티의 영속성 상태 변화를 자식 엔티티에 모두 전이|

영속성 전이는 단일 엔티티에 완전히 종속적이고 부모 엔티티와 자식 엔티티의 라이프 사이클이 유사할 때 활용하는 것을 추천한다.

***고아 객체 제거하기***

부모 엔티티와 연관 관계가 끊어진 자식 엔티티를 고아 객체라고 한다.

고아 객체 제거 기능은 참조하는 곳이 하나일 때만 사용해야는데 다른 곳에서도 참조하고 있는 엔티티인데 삭제하면 문제가 생길 수 있다.
주문 상품 엔티티를 주문 엔티티가 아닌 다른 곳에서 사용하고 있다면 이 기능을 사용하면 안 된다.

```
package com.shop.entity;

import ...

@Entity
public class Order {
    ...
    
    @OneToMany(mappedBy = "order", cascade = ...., orphanRemoval = true)
    private List<OrderItem> orderItems = new ArrayList<>();
}
```
