# 컬렉션 조회 최적화

- XXXtoMany라면, List로 반환하는 경우가 존재한다. 이때, 발생가능한 문제점들에 대해서 알아보고 해결방법을 찾아 적용해보자.

---

- 아래의 코드에서 가질 수 있는 문제는 무엇일까?

```java
@GetMapping("/api/v1/orders")
public List<Order> ordersV1() {
    List<Order> all = orderRepository.findAllByString(new OrderSearch());
    for (Order order : all) {
        order.getMember().getName(); //Lazy 강제 초기화
        order.getDelivery().getAddress(); //Lazy 강제 초기화
        List<OrderItem> orderItems = order.getOrderItems();
        orderItems.stream().forEach(o -> o.getItem().getName()); //Lazy 강제 초기화

    }
    return all;
}
```

1. 엔티티를 직접 노출시키고 있다. Api spec을 직접 노출하고 있으며 엔티티에 의존하고 있다. ⇒ 운영과정 및 유지보수 문제

사실, 엔티티 노출 문제는 컬렉션일때만 발생가능한 문제가 아니다.

1. 지연로딩 처리 이후, 강제 초기화 작업을 계속 진행해주기에, query가 어마어마하게 많이 나가게 된다. ⇒ 성능문제

사실, 해당 성능문제는 컬렉션일때만 발생하는 문제가 아니다. toOne일때도 고려해야함. → fetch join

---

## 엔티티를 DTO로 변환

OrderService 관련 API를 만드는 과정에서 엔티티 노출을 시키지 않게 하기 위해서 “DTO”를 활용한다.

다만, 주의해야할 점이 존재한다.

- OrderItems의 value들이 역시 Entity이라는 것도 고려해야한다.

주의사항) DTO를 만드는 과정에서 OrderItem도 DTO로 만들어줘야한다는 것이다.

```java
@Data
static class OrderDto {
    private Long orderId;
    private String name;
    private LocalDateTime orderDate; //주문시간
    private OrderStatus orderStatus;
    private Address address;
    private List<OrderItemDto> orderItems;
    public OrderDto(Order order) {
        orderId = order.getId();
        name = order.getMember().getName();
        orderDate = order.getOrderDate();
        orderStatus = order.getStatus();
        address = order.getDelivery().getAddress();
        orderItems = order.getOrderItems().stream()
                .map(orderItem -> new OrderItemDto(orderItem))
                .collect(toList());
    }
}

@Data
static class OrderItemDto {
    private String itemName;//상품 명
    private int orderPrice; //주문 가격
    private int count; //주문 수량
    public OrderItemDto(OrderItem orderItem) {
        itemName = orderItem.getItem().getName();
        orderPrice = orderItem.getOrderPrice();
        count = orderItem.getCount();
    }
}
```

---

## fetch join적용

query가 총 한번만 나가도록 fetch join을 사용해서 성능 최적화를 진행해줘야한다.

```java
@GetMapping("/api/v3/orders")
public List<OrderDto> ordersV3() {
    List<Order> orders = orderRepository.findAllWithItem();
    List<OrderDto> result = orders.stream()
            .map(o -> new OrderDto(o))
            .collect(toList());
    return result;
}
```

```java
public List<Order> findAllWithItem() {
    return em.createQuery(
                    "select distinct o from Order o" +
                            " join fetch o.member m" +
                            " join fetch o.delivery d" +
                            " join fetch o.orderItems oi" +
                            " join fetch oi.item i", Order.class)
            .getResultList();
}
```

- DB data의 뻥튀귀가 join과정에서 발생할 수 있다는 문제가 존재한다.
    
    1:다  ⇒ 다 만큼의 개수로 생성되는 DB join rule → **distinct**를 활용한다.
    
- 주의사항
    
    sql에서의 distinct의 기능에서 중복제거는 완전히 똑같은 값만 날려주기에, id값이 같더라도 중복제거가 되지 않는 문제가 발생가능하다.
    
    이때, jpa가 제공하는 **distinct**를 사용하면 된다.
    

✅ JPA가 제공하는 distinct는 join하는 기준만 비교해서 중복제거를 추가로 진행해준다.

과정요약 ) DB에 distinct를 날리고, 엔티티가 중복인 경우 걸러서 클래스에 담아준다.

---

## ‘1:다’에서의 fetch join의 제약사항과 단점

- 페이징 조건을 memory에서 처리한다. ⇒ 이로인한 성능문제 ( Out of Memroy)

필요한 것을 모두 불러온다음에 memory에서 처리하게 된다.

페이징이 불가능한 이유

위에서 언급한, data 뻥튀기로 인해서 offset이 2일때, 3번째 객체부터 불러오고 싶었던 것인데, 뻥튀기 되어있는 이전의 객체가 불려와지는 경우가 존재할 수 있게 되는 것이다. 따라서 정확한 페이징이 불가능하기에 이러한 문제를 일으키지 않기 위해, 하이버네이트는 memory에서 처리하도록 세팅되어있다. 

다만 이로인해 성능문제가 발생한다.

- 컬렉션 패치 조인은 1개만 사용이 가능하다. 둘 이상에 패치조인이면 꼬여버려서 처리를 제대로 하지 못하게 될 수 있다.

그렇다면, 해결책은?

---

## 페이징 한계 돌파

- 1:1 관계는 fetch join을 그냥 사용하면 됨. data 뻥튀기가 발생하지 않기에.
- 1:다 관계 → lazy로 놓고 **in query**를 활용해서 N을 1로 변경

**변경적용**

1. before

```java
public List<Order> findAllWithItem() {
      return em.createQuery(
                      "select distinct o from Order o" +
                              " join fetch o.member m" +
                              " join fetch o.delivery d" +
                              " join fetch o.orderItems oi" +
                              " join fetch oi.item i", Order.class)
              .getResultList();
  }
```

1. after

```java
public List<Order> findAllWithMemberDelivery(int offset, int limit) {

      return em.createQuery(
      "select o from Order o" +
              " join fetch o.member m" + // 1:1관계
              " join fetch o.delivery d", Order.class) //1:1관계
              .setFirstResult(offset)
              .setMaxResults(limit)
              .getResultList();
  }
```

1:N의 문제가 발생할 것이다. 이때, 어떻게 성능 튜닝을 할 수 있을까?

**N을 1로 변경**

option → in query의 개수를 설정한다. ⇒ default_batch_fetch_size을 사용한다

(default_batch_fetch_size는 최대 1000까지 가능)

- yaml → global 적용

```java
default_batch_fetch_size: 100
```

- @BatchSize annotation 활용 → 개별적용

**특징**

**in query**를 사용하면 컬렉션을 하나씩 여러번 쿼리를 날려서 가져오지 않고 설정값만큼 한번의 쿼리에 의해서 가져오는 효과를 볼 수 있다.

현재 사용자별로 itemOrder가 2번씩 존재해서 총 4개가 존재하는데, 이것도 역시 한번에 가져온다.

1:N:N ⇒ 1:1:1로 변경해준다.

**효과**

 paging도 가능하고 data뻥튀기가 된 채로 넘어오지 않기에 traffic을 줄일 수 있다.

그냥 fetch만 사용했을 때보다 query가 더 많기는 하지만, 1:1:1로 변경이 가능해서 어느정도 문제를 해결가능하다.

✅ 1:다의 관계에서 paging을 하기 위해서, **In query**를 사용해서 한번에 필요한것을 설정단위만큼 한번에 가져와야한다.

주의) 1000까지 가능.

순간적인 DB 부하가 올 수 있어서 1000까지 높을 수 있다는 것도 고려해야한다. 다만, 100이든 1000이든 메모리 입장에서는 최종적으로 전체데이터를 로딩하는 것은 동일하기에 메모리 사용량은 동일하다.

---

## JPA에서 DTO 직접 조회

위에서는 collections에 대해서 sql을 처리하는 `findAllWithMemberDelivery` 을를 OrderRepository에서 처리하고 있음.

**그렇다면, sql과 OrderRepository를 분리해서 역할의 분리가 될 것이기에, query를 처리하는 sql을 따로 분리해준다.**

→ sql을 따로 처리해주는 DTO를 생성해주자!

또, 원하는 속성만 처리하기 위해서 DTO를 만들어야한다.

- OrderQueryRepository를 만들어서 처리해준다.
1. entity manager 사용
2. DTO를 불러와서 처리 → 이것도 OrderQueryDto로 따로 생성 후 사용

**OrderQueryDto**

```java
@Repository
@RequiredArgsConstructor
public class OrderQueryRepository {
    private final EntityManager em;
	/**
	* 컬렉션은 별도로 조회
	* Query: 루트 1번, 컬렉션 N 번 * 단건 조회에서 많이 사용하는 방식 */
	public List<OrderQueryDto> findOrderQueryDtos() { //루트 조회(toOne 코드를 모두 한번에 조회)
	        List<OrderQueryDto> result = findOrders();
	//루프를 돌면서 컬렉션 추가(추가 쿼리 실행) result.forEach(o -> { 
	            List<OrderItemQueryDto> orderItems = findOrderItems(o.getOrderId());
	            o.setOrderItems(orderItems);
	        });
	        return result;
	    }
	
	/**
	* 1:N 관계(컬렉션)를 제외한 나머지를 한번에 조회
	*/
	private List<OrderQueryDto> findOrders() {
			return em.createQuery(
			                "select new jpabook.jpashop.repository.order.query.OrderQueryDto(o.id, m.name, o.orderDate, o.status, d.address)" +
							        " from Order o" +
							        " join o.member m" +
							        " join o.delivery d", OrderQueryDto.class)
			.getResultList();
		}
	
	
		/**
		* 1:N 관계인 orderItems 조회
		*/
	  private List<OrderItemQueryDto> findOrderItems(Long orderId) {
		        return em.createQuery(
		                "select new jpabook.jpashop.repository.order.query.OrderItemQueryDto(oi.order.id, i.name, oi.orderPrice, oi.count)" +
		                        " from OrderItem oi" +
		                        " join oi.item i" +
		                        " where oi.order.id = : orderId",
		OrderItemQueryDto.class).setParameter("orderId", orderId)
	} 
}
```

**OrderItemDto**

```java
@Data
@EqualsAndHashCode(of = "orderId")
public class OrderQueryDto {
private Long orderId;
private String name;
private LocalDateTime orderDate; //주문시간 private OrderStatus orderStatus;
private Address address;
private List<OrderItemQueryDto> orderItems;
      public OrderQueryDto(Long orderId, String name, LocalDateTime orderDate,
		  OrderStatus orderStatus, Address address) {
          this.orderId = orderId;
          this.name = name;
          this.orderDate = orderDate;
          this.orderStatus = orderStatus;
          this.address = address;
} }
```

 

**OrderQueryDto**

```java
@Data
public class OrderItemQueryDto {
		@JsonIgnore
		private Long orderId; //주문번호 private 
		String itemName;//상품 명 
		private int orderPrice; //주문 가격
		private int count; //주문 수량
		      public OrderItemQueryDto(Long orderId, String itemName, int orderPrice, int count) {
		          this.orderId = orderId;
		          this.itemName = itemName;
		          this.orderPrice = orderPrice;
		          this.count = count;
} }
```

- 최초 query가 1번 → N개의 쿼리가 날려진다는 문제가 존재

---

## 최적화를 적용한 DTO 직접 조회 - 컬렉션 조회 최적화 진행

- order에 대한 것을 루프를 돌면서 N번의 query가 나가는 것을 변경
    
    → order를 in query로 1번 조회하도록 한다.
    

**in query**를 적용

```java
@GetMapping("/api/v5/orders")
public List<OrderQueryDto> ordersV5() {
    return orderQueryRepository.findAllByDto_optimization();
}
```

- in query처리

```java
public List<OrderQueryDto> findAllByDto_optimization() { 
		//루트 조회(toOne 코드를 모두 한번에 조회)
    List<OrderQueryDto> result = findOrders();
		//orderItem 컬렉션을 MAP 한방에 조회
    Map<Long, List<OrderItemQueryDto>> orderItemMap = findOrderItemMap(toOrderIds(result));
		//루프를 돌면서 컬렉션 추가(추가 쿼리 실행X)
		result.forEach(o -> o.setOrderItems(orderItemMap.get(o.getOrderId())));
		    return result;
}

private List<Long> toOrderIds(List<OrderQueryDto> result) {
    return result.stream()
            .map(o -> o.getOrderId())
            .collect(Collectors.toList());
}

private Map<Long, List<OrderItemQueryDto>> findOrderItemMap(List<Long> orderIds) {
    List<OrderItemQueryDto> orderItems = em.createQuery(
            "select new jpabook.jpashop.repository.order.query.OrderItemQueryDto(oi.order.id, i.name, oi.orderPrice, oi.count)" +
                    " from OrderItem oi" +
                    " join oi.item i" +
                    " **where oi.order.id in :orderIds**", OrderItemQueryDto.class)
            .setParameter("orderIds", orderIds)
            .getResultList();
				    return orderItems.stream().collect(Collectors.groupingBy(OrderItemQueryDto::getOrderId));
}
```

`where [oi.order.in](http://oi.order.in) in :orderIds` jap query 구문을 in query로 만들어 orderid가 여러개라면 한번에 쿼리를 작성해서 가져오도록한다.

현재 2개의 주문이 존재하고 따라서 2개의 쿼리가 총 1번의 쿼리로 최적화되어 1+N의 문제를 해결

- 차이점

in query 사용 전에는 루프를 돌면서 query가 나갔지만, 현재는 query를 1번 돌리고 Map을 사용해서 메모리에 저장시켜 사용하는 방식을 통해 query성능 최적화를 시킴

- trade off

장점) collection fetch join보다 원하는 값만 쿼리하기 위해 select → 데이터가 줄어드는 장점이 존재

단점 ) fetch join보다 복잡.

---

## 위의 버전을 더욱 최적화

**아이디어**

- 어떻게 하면 1+1이 아닌 한번의 query를 통해서 다 가져올 수 있나?
- 두개를 DB에서 join하면 collection이 아닌 것과 collection인 속성들에 대해서 따로 가져올 필요가 없이 전부를 한번에 가져오면 된다.
- 그렇다면 한번에 가져오기 위해서 어떤 과정을 거쳐야하나?에 대해서 알아야함.

- OrderFlatDto

Order와 OrderItem을 join해서 한번에 가져오기 

⇒ 2개의 DTO를 어떻게 변경시켜야하나? ( 따로 정의도이었던 DTO의 속성 값들을 한번에 정의하고 그것들을 한번에 가져오겠다는 것을 의미)

```java
@Data
public class OrderFlatDto {
	private Long orderId;
	private String name;
	private LocalDateTime orderDate; //주문시간 
	private Address address;
	private OrderStatus orderStatus;

	private String itemName;//상품 명 
	private int orderPrice; //주문 가격 
	private int count; //주문 수량

  public OrderFlatDto(Long orderId, String name, LocalDateTime orderDate,
  OrderStatus orderStatus, Address address, String itemName, int orderPrice, int
  count) {
      this.orderId = orderId;
      this.name = name;
      this.orderDate = orderDate;
      this.orderStatus = orderStatus;
      this.address = address;
      this.itemName = itemName;
      this.orderPrice = orderPrice;
      this.count = count;
		}
}
```

- OrderQueryRepository

sql을 처리하는 query에 대한 repository → em.CreateQuery를 작성

```java
public List<OrderFlatDto> findAllByDto_flat() {
      return em.createQuery(
              "select new jpabook.jpashop.repository.order.query.OrderFlatDto(o.id, m.name, o.orderDate,o.status, d.address, i.name, oi.orderPrice, oi.count)" +
                     " from Order o" +
                      " join o.member m" +
                      " join o.delivery d" +
                      " join o.orderItems oi" +
                      " join oi.item i", OrderFlatDto.class)
              .getResultList();
}
```

**연관관계에 있는 모든 엔티티를 조인** 시켜서 모두 합쳐줌.

**다만**, 그렇다면 다:1 관계에서는 다에 맞추기에 1이 중복으로 들어가서 Order를 기준으로 가져와도(현재 준비된 데이터의 경우, 2개) 총 4개(1쪽이 Order이고 orderItems가 각각 2개)를 가져옴.

- OrderApiController

```java
@GetMapping("/api/v6/orders")
public List<OrderQueryDto> ordersV6() {
    List<OrderFlatDto> flats = orderQueryRepository.findAllByDto_flat();
    return flats.stream()
            .collect(groupingBy(o -> new OrderQueryDto(o.getOrderId(),
					  o.getName(), o.getOrderDate(), o.getOrderStatus(), o.getAddress()),
             mapping(o -> new OrderItemQueryDto(o.getOrderId(),
						  o.getItemName(), o.getOrderPrice(), o.getCount()), toList())
            )).entrySet().stream().map(e -> new OrderQueryDto(e.getKey().getOrderId(),
					  e.getKey().getName(), e.getKey().getOrderDate(), e.getKey().getOrderStatus(),
					  e.getKey().getAddress(), e.getValue()))
            .collect(toList());
}
```

    메모리에서 처리하기 때문에 query에 영향은 없는 코드

    역할 : FlatDto to OrderQueryDto으로 api spec을 변경하는 코드 ( 한줄로 나온 것을 직접 분해하여서 원하는대로 사용하면 된다)

- OrderQueryDto

```java
public OrderQueryDto(Long orderId, String name, LocalDateTime orderDate,
  OrderStatus orderStatus, Address address, List<OrderItemQueryDto> orderItems) {
      this.orderId = orderId;
      this.name = name;
      this.orderDate = orderDate;
			this.orderStatus = orderStatus;
      this.address = address;
      this.orderItems = orderItems;
}
```

- trade off

    쿼리가 줄어들긴 하지만, 단점은 애플리케이션에서의 코드 조작을 통해 스펙을 직접 맞춰줘야함.

    데이터가 클 때, 1:다에서 다에 맞추기 때문에 Order를 기준으로 페이징이 불가능.

    (다만 OrderItem에 대해서는 가능하겠지만!)