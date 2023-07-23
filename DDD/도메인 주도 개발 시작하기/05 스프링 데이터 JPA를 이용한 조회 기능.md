# 검색을 위한 스펙

검색 조건이 고정되어 있거나 단순할 경우 특정 조건으로 조회하는 기능

~~~java
public interface OrderDataDao {
    Optional<OrderData> findById(OrderNo id);
    List<OrderData> findByOrderer(String ordererId, Date fromDate, Date toDate);
    // ...
}
~~~

find 메서드에 필요한 컬럼을 연속으로 조합해서 사용할 수 있다.

<br>

👉 **필요한 조건이 많아지면 많아질수록 정의해야할 컬럼들이 많아져 find 메서드명이 길어진다. 검색 조건을 다양하게 조합해야 할 때 사용할 수 있는 것이 스펙(Specification)이다.**

- 애그리거트가 특정 조건을 충족하는 지 검사할 때 사용하는 인터페이스이다.

<br>

**Specification 정의** 

~~~java
public interface Specification<T> {
    public boolean isSatisfiedBy(T agg);
}
~~~

isSatisFiedBy() 메서드는 검사 대상 객체가 조건을 충족하면 true, 아닐 경우 false를 리턴한다.

agg 파라미터는 검색 대상이 되는 객체다.

- 스펙을 Repository에서 사용할 경우 : agg는 애그리거트 루트이다.
- 스펙을 DAO에서 사용할 경우 : agg는 검색 결과로 리턴할 데이터 객체이다.

<br>

**Order 애그리거트 객체가 특정 고객의 주문인 지 확인하는 스펙 구현**

~~~java
public class OrdererSpec implements Specification<Order> {
    private String ordererId;

    public OrdererSpec(String orderId) {
        this.ordererId = ordererId;
    }

    public boolean isSatisFiedBy(Order agg) {
        return agg.getOrdererId().getMemberId().getId().equals(ordererId);
    }
}
~~~

Repository나 DAO는 검색 대상을 걸러내는 용도로 스펙을 사용

<br>

**메모리에 모든 애그리거트를 보관하고 있는 조건이 있을 때 스펙 사용**

~~~java
public class MemoryOrderRepository implements OrderRepository {
    public List<Order> findAll(Specification<Order> spec) {
        List<Order> allOrders = findAll();
        return allOrders.stream()
                .filter(order -> spec.isSatisfiedBy(order)).toList();
    }
    // ...
}
~~~

<br>

**원하는 스펙을 생성해 리포지토리에 전달**

~~~java
// 검색 조건을 표현하는 스펙 생성
Specification<Order> ordererSpec = new OrdererSpec("condition_value");
// 리포지토리에 전달
List<Order> orders = orderRepository.findAll(ordererSpec);
~~~

하지만 실제 스펙은 위와 같이 구현하지 않는다. 메모리에 모든 객체를 보관하는 것도 어렵고, 가능하더라도 조회하는데 성능 이슈가 있기 때문이다.

<br>

# 스프링 데이터 JPA를 이용한 스펙 구현

**스프링 데이터 JPA의 Specification의 정의**

~~~java
public interface Specification<T> extends Serializable {
    // where, and, or, not 메서드 등
    
    @Nullable
    Predicate toPredicate(Root<T> root,
                          CriteriaQuery<?> query,
                          CriteriaBuilder db);
}
~~~

- 지네릭 파입 파라미터 T : JPA 엔티티 타입을 의미
- toPredicate() : 조건을 표현하는 Predicate를 생성

<br>

**스펙 인터페이스를 구현한 클래스**

~~~java
public class OrdererIdSpec implements Specification<OrderSummary> {
    private String ordererId;

    public OrdererIdSpec(String ordererId) {
        this.ordererId = ordererId;
    }

    @Override
    public Predicate toPredicate(Root<OrderSummary> root,
                                 CriteriaQuery<?> query,
                                 CriteriaBuilder cb) {
        return cb.eqauls(root.get(OrderSummary_.ordererId), ordererId);
    }
}
~~~

- OrderSummary에 대한 검색 조건을 구현

- ordererId 프로퍼티 값이 생성자로 전달 받은 ordererId와 동일한 지 비교하는 Predicate 생성

- OrderSummary_는 JPA 정적 메타 모델을 정의한 코드이다.

<br>

**스펙 구현 클래스를 개별적으로 만들지 않고 스펙 생성 기능을 별도로 모은 클래스 **

~~~java
public class OrderSummarySpecs {
    public static Specification<OrderSummary> orderId(String odererId) {
        return (Root<OrderSummary> root, CriteriaQuery<?> query,
        CriteriaBuilder cb) ->
                cb.equals(root.<Sting>get("ordererId"), ordererId);
    }

    public static Specification<OrderSummary> orderDateBetween(LocalDateTime from, LocalDateTime to) {
        return (Root<OrderSummary> root, CriteriaQuery<?> query,
        CriteriaBuilder cb)->
                cb.between(root.get(OrderSummary_.orderDate), from, to);
    }
}
~~~

- 스펙 인터페이스는 함수형 인터페이스이기 때문에 람다식을 이용해 객체를 생성

- 스펙 생성 기능을 별도로 모은 클래스를 정의함으로써 호출하는 쪽에서는 조금 더 간결하게 스펙 생성을 할 수 있음

<br>

**스펙 생성 기능을 호출하는 코드**

~~~java
Specification<OrderSummary> betweenSpec = OrderSummarySpecs.orderDateBetween(from, to);
~~~

<br>

# 리포지토리/DAO에서 스펙 사용하기

findAll() 메서드를 호출해서 스펙에 충족하는 엔티티를 검색할 수 있다.

~~~java
public interface OrderSummaryDao extends Repository<OrderSummary, String> {
    List<OrderSummary> findAll(Specification<OrderSummary> spec);
}
~~~

**스펙 구현체를 사용한 엔티티 검색**

~~~java
// 스펙 객체 생성
Specification<OrderSummary> spec = new OrdererIdSpec("user1");
// findAll() 메서드를 이용한 검색
List<OrderSummary> results = orderSummaryDao.findAll(spec);
~~~

<br>

# 스펙 조합

스펙 인터페이스는 and(), or(), not(), where() 등의 메서드를 제공한다.

### and(), or()

~~~java
public interface Specification<T> extends Serializable {
    default Specification<T> and(@Nullable Specification<T> other) {...}
    default Specification<T> or(@Nullable Specification<T> other) {...}

    @Nullable
    Predicate toPredicate(Root<T> root,
                          CriteriaQuery<?> query,
                          CriteriaBuilder criteriaBuilder);
}
~~~

- and() : 두 스펙을 모두 충족하는 조건을 표현하는 스펙 생성
- or() : 두 스펙 중 하나 이상 충족하는 조건을 표현하는 스펙 생성

<br>

**and()를 사용한 예시**

~~~java
Specification<OrderSummary> spec = OrderSummarySpecs.ordererId("user1")
        .and(OrderSummarySpecs.orderDateBetween(from, to));
~~~

<br>

### **not()**

~~~java
public static <T> Specifications<T> not(Specification<T> spec)
~~~

- not() : 조건을 반대로 적용할 때 사용

<br>

**not()를 사용한 예시**

~~~java
Specification<OrderSummary> spec = Specification.not(OrderSummarySpecs.ordererId("user1"));
~~~

<br>

### **where()**

~~~java
Specification<OrderSummary> spec = Specification.where(createNullableSpec()).and(createOtherSpec());
~~~

- where() : 파라미터로 null을 전달하면 아무 조건도 생성하지 않는 스펙 객체를 리턴
  - spec 객체의 NPE 문제를 고민하지 않아도 된다.

<br>

# 정렬 지정하기

