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

스프링 데이터 JPA의 두 가지 정렬 방법

- 메서드 이름에 OrderBy를 사용한 정렬
- Sort를 인자로 전달

<br>

**메서드 이름을 활용한 정렬**

~~~java
public interface OrderSummaryDao extends Repository<OrderSummary, String> {
    List<OrderSummary> findByOrdererIdOrderByNumberDescNumberAsc(String ordererId);
}
~~~

- 정렬에 필요한 프로퍼티가 두 개 이상이면 메서드 이름이 길어지는 단점이 있음
- 메서드 이름으로 정렬 순서를 정하기 때문에 차후에 정렬 순서를 변경하려면 새로운 메서드를 생성해야 하거나 수정하기 어려운 상황이 있을 수 있음

<br>

**Sort 타입을 파라미터로 전달을 통한 정렬**

~~~java
public interface OrderSummaryDao extends Repository<OrderSummary, String> {
    List<OrderSummary> findByOrdererId(String ordererId, Sort sort);
}

Sort sort = Sort.by("number").ascending().and(Sort.by("orderDate").descending());
List<OrderSummary> result = orderSummaryDao.findByOrdererId("user1", sort);
~~~

<br>

# 페이징 처리하기

스프링 데이터 JPA는 페이징 처리를 위해 Pageable 타입을 이용

~~~java
public interface MemberDataDao extends Repository<MemberData, String> {
    List<MemberData> findByNameLike(String name, Pageable pageable);
}
~~~

- Pageable은 인터페이스이고, 실제 구현 객체는 PageRequest이다.

<br>

**구현 객체인 PageRequest와 Sort를 활용한 정렬된 페이징 처리**

~~~java
Sort sort = Sort.by("name").descending();
PageRequest pageReq = PageRequest.of(1, 2, sort);
List<MemberData> user = memberDataDao.findByNameLike("사용자%", pageReq);
~~~

<br>

**리턴 타입을 Page로 활용한 전체 데이터 개수(count query) 등의 페이징 처리에 필요한 데이터 조회**

~~~java
Pageable pageReq = PageRequest.of(2, 3);
Page<MemberData> page = memberDataDao.findByBlocked(false, pageReq);

List<MemberData> content = page.getContent(); // 조회 결과 목록
long totalElements = page.getTotalElements(); // 조건에 해당하는 전체 개수
int totalPages = page.getTotalPages(); // 전체 페이지 번호
~~~

❗**주의할 점**

- 리턴 타입이 List면 COUNT 쿼리는 실행되지 않는다. count 등 페이징 처리와 관련된 정보가 필요하지 않을 경우, List 리터 타입을 사용하면 된다.
- 반면에 **스펙을 사용하는 findAll 메서드에 Pageable 타입을 사용하면 리턴 타입이 Page가 아니여도 COUNT 쿼리가 실행된다. **
- 스펙을 사용하고 페이징 처리를 하면서 COUNT 쿼리는 실행하고 싶지 않을 경우, 커스텀 리포지토리 기능을 직접 구현하면 된다.

<br>

**N개 데이터를 조회하는 findFirstN 메서드**

~~~java
List<MemberData> findFirst5ByNameOrderByName(String name);
~~~

- 정렬된 데이터에서 처음 5개의 데이터를 조회
- First 대신 Top을 사용해도 된다.
- First 또는 Top 뒤에 숫자가 없으면 한 개의 결과만 조회

<br>

# 스펙 조합을 위한 스펙 빌더 클래스

**조건에 따라 스펙을 조합하는 코드**

~~~java
Specification<MemberData> spec = Specification.where(null);
if (searchRequest.isOnlyNotBlocked()) {
    spec = spec.and(MemberDataSpecs.nonBlocked());
}
if (StringUtils.hasText(searchRequest.getName())) {
    spec = spec.and(MemberDataSpecs.nameLike(searchRequest.getName()));
}

List<MemberData> results = memberDataDao.findAll(spec, PageRequest.of(0, 5));
~~~

<br>

**위와 동일한 조건을 스펙 빌더로 사용한 코드**

~~~java
Specification<MemberData> spec = SpecBuilder.builder(MemberData.class)
        .ifTrue(searchRequest.isOnlyNotBlocked(),
                () -> MemberDataSpecs.nonBlocked())
        .ifHasText(searchReqeust.getName(),
                name -> MemberDataSpecs.nameLike(searchRequest.getName()))
        .toSepc();

List<MemberData> results = memberDataDao.findAll(spec, PageRequest.of(0, 5));
~~~

- if문을 사용하여 조건 체크를 하는 것보다 스펙 빌더를 사용한 코드가 호출 체인으로 연속된 변수 할당을 줄일 수 있어 조금 더 가독성이 높고 구조가 단순하다.
- 스펙 빌더는 and(), ifHasText(), ifTrue() 메서드를 제공하고, 이 외에 필요한 메서드는 추가해서 사용할 수 있다.

<br>

# 동적 인스턴스 생성

JPA는 쿼리 결과에서 임의의 객체를 동적으로 생성할 수 있는 기능을 제공한다.

<br>

**JPQL을 활용한 동적 인스턴스 생성 코드**

~~~java
public interface OrderSummaryDao extends Repository<OrderSummary, String> {
    @Query("""
            select new com.myshop.order.query.dto.OrderView(
                o.number, o.state, m.name, m.id, p.name
            )
            from Order o join o.orderLines ol, Member m, Product p
            where o.orderer.memberId.id = :orderId
            and o.orderer.memberId.id = m.id
            and index(ol) = 0
            and ol.productId.id = p.id
            order by o.number.number desc
            """)
    List<OrderView> findOrderView(String ordererId);
}
~~~

- select 절에 new 키워드와 패키지명을 포함한 클래스명을 지정하고 괄호안에 생성자에 인자로 전달할 값들을 지정하면 해당 객체의 생성자를 통해 인스턴스를 생성할 수 있다.

<br>

💁‍♂️ **동적 인스턴스 생성의 장점**

- JPQL을 그대로 사용하므로 객체 기준으로 쿼리를 작성할 수 있으며, 지연 및 즉시 로딩에 대한 고민없이 원하는 데이터를 조회할 수 있다.

<br>

# 하이버네이트 @Subselect 사용

하이버네이트는 JPA 확장 기능으로 @Subselect를 제공한다.

**@Subselect는 쿼리 결과를 @Entity로 매핑할 수 있는 기능이다.**

<br>

**@Subselect를 이용한 @Entity 매핑 예**

~~~java
@Entity
@Immutable
@Subselect("""
            select o.order_number as number,
            o.version, o.ordererId, o.orderer_name,
            o.total_amounts,
            p.product_id, p.name as product_name
            from purcharse_order o inner join order_line ol
            on o.order_number = ol.order_number
            cross join product p
            where
            ol.line_idx = 0
            and ol.product_id = p.product_id
            """
)
@Synchronize({"purcharse_order", "order_line", "product"})
public class OrderSummary {
    @Id
    private String number;
    private long version;
    @Column(name = "orderer_id")
    private String ordererId;
    @Column(name = "orderer_name")
    private String ordererName;
    //...
    
    protected OrderSummary() {}
}
~~~

하이버네이트 전용 어노테이션인 @Immutable, @Subselect, @Synchronize을 사용해서 테이블이 아닌 쿼리 결과를 @Entity에 매핑할 수 있다.

- @Subselect : 쿼리 실행 결과를 매핑할 테이블처럼 사용
  - 매핑된 Entity는 수정할 수 없다. 만약 수정을 한다면 매핑한 테이블이 없으므로 에러가 발생한다. 이 문제를 방지하기 위해 @Immutable을 사용
- @Immutable : Entity의 매핑 필드/프로퍼티가 수정되어도 DB에 반영하지 않고 무시한다.
- @Synchoronize : Entity를 로딩하기 전에 지정한 테이블이 변경되었을 경우, Flush 처리를 해준다.
  - 설정하지 않을 경우, Entity에서 상태 변경한 뒤 조회를 할 경우, 변경된 상태가 반영이 되지 않은 상태에서 조회가 될 수 있다.

<br>

👉 @Subselect는 지정한 쿼리를 from 절의 서브 쿼리로 사용한다. 서브 쿼리를 사용하고 싶지 않다면 네이티브 SQL 쿼리를 사용하거나 Mybatis와 같은 별도의 매퍼를 사용해서 조회 기능을 구현할 수 있다.

~~~sql
select a.product_id
from (
        // 지정한 쿼리
        select p.product_id
        from product p
        where p.product_id
) a
~~~

