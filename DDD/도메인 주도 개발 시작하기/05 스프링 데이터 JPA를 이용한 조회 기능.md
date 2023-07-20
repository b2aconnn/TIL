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

