# JPA

## JPA란 무엇인가?

자바 진영의 ORM 기술 표준이다.
<br>

하이버네이트가 엔티티빈(EJB)가 쓸모없어서 더 발전시킨 디비 접근 기술을 고안하며 생성됨.<br>


- **하이버네이트** 등의 구현체를 사용한 인터페이스가 만들어졌는데 이를 JPA라고 함


------

## sql 중심적인 개발의 문제점 = ORM이 필요한 이유

### sql 의존성

- 코드 변경의 경우<br>
sql을 직접 작성할 때는, 필드가 추가됨과 동시에 sql 쿼리들에 접근해서 컬럼명을 추가하하는 등의 코드를 변경해야한다.<br>

### 불편함

- sql mappper<br>
각 기능에 필요한 쿼리들을 작성하고 원하는 데이터를 가져오는 역할을 "개발자"가 하게 되어 개발자가 sql mapper가 된다.<br>


### 객체 모델링

객체 모델링에서의 차이를 가지는데 "상속"관계를 보자.<br>
객체와 관계형 데이터베이스의 차이점이 존재한다.<br>

객체 사이에서는 상속관계가 존재하지만, 사실 table끼리는 상속이라기보다 슈퍼타입 서브타입 관계이다.<br>
table은 **상속관계가 존재하지 않는다.**

![image](https://user-images.githubusercontent.com/74058047/222360053-1cef628a-abbe-46a1-8ad9-51c63379e4cd.png)

1. 조회를 하는 경우<br>
**sql**<br>
item과 album을 join하여서 데이터를 가져와야한다.
<br>

**object**<br>
album의 정보를 알고싶다면 object의 경우 collection을 활용하면 된다. **다형성**을 사용해도 되고 **참조**를 사용하면 된다.<br> 

2. update를 하는 경우<br>

**sql**<br>
정보를 저장하고 싶을 때 album에도 접근하고 item에도 접근하여 select query 2번을 작성하여 데이터를 가져온다.

**object**<br>
java collection을 사용해서 쉽게 데이터를 변경할 수 있다.<br>


### 객체를 테이블에 맞춰서 모델링

연관관계를 작성할 때, 객체 모델링은 테이블의 정보를 보고 작성한다.<br>

![image](https://user-images.githubusercontent.com/74058047/222361469-0415de93-1052-47d6-b49b-03518667fc9d.png)
<br>
다음과 같이 객체에 다른 객체를 넣어 참조를 나타낼 때, sql로 작성하게 되면 id값을 참조하는 객체에 접근해서 가져오게 되는 과정을 거쳐야한다.<br> 
sql이 아닌 객체 모델링 즉 자바 컬렉션을 통해서 관리한다면 어떻게 될까?<br>

- collection으로 관리하기

```
list.add(member);

Member member = list.get(memberId);
Team team = member.getTeam();
```
다음과 member객체에서 team을 getTeam()을 통해서 접근할 수 있다.<br>

### entity 신뢰의 문제

**Sql**<br>
처음 실행하는 sql에 따라서 탐색 범위가 결정된다.<br>

![image](https://user-images.githubusercontent.com/74058047/222361469-0415de93-1052-47d6-b49b-03518667fc9d.png)

따라서, 여기서는 getOrder()를 했을 때 `null`이 나오게 된다.<br>

이는 신뢰의 문제로 이어진다.<br>
**객체는 자유롭게 객체 그래프를 탐색할 수 있어야하는데 어디까지 이어져있는지 개발자가 sql을 분석해서 직접 찾아줘야하고 이 과정에서 실수가 발생이 가능하다.**<br>

그렇다면 다중 join을 사용해서 select하면 모두 연결되지 않을까?<br>
1~3개가 아니라 매우 많다면 불가능에 가깝다고 효율성도 나쁘고 불가능에 가깝다고 보면된다.<br>

이러한 sql의 신뢰의 문제가 발생하는 것은 물리적으로 계층이분리되어있지만, 결국 분리되지 않은 것처럼 DAO를 찾아서 sql을 분석하기에 논리적으로는 분석되어있지 않다고한다.<br>

### 비교할때의 문제점

- sql을 사용할 경우 -> 같은 id여도 생성된 인스턴스끼리 비교하기에 다른게 나온다.<br>
- java collection을 사용할 경우 -> key, value를 통해서 접근하기에 key가 같다면 value는 동일하다.<br>

### 결론 

객체를 sql을 통해서 복잡하게 조회하지 않고 저장하지 않고 repository에서 자바 collection을 사용하여 데이터를 다루듯이 컬렉션을 통해서 처리하는 방법을 탐색하기 시작했다.
<br>
ORM이 있다면 편리하게 조회 및 저장이 가능하고 sql로 접근했을 때의 문제점을 해결해준다.<br>


------

## JPA의 효과

- 객체 중심적인 개발<br>
- 생산성 => JPA와 CRUD를 살펴보면 단순함.<br>
저장 : jpa.persist(member)

조회 : jpa.find(memberid)

수정 : member.setName(”name”)

삭제 : jap.remove(member)<br>
- 유지보수<br>
- 성능최적화<br>

1. 1차 캐시와 동일성 보장<br>
```
String memberId = "100";
//1
Member m1 = jpa.find(Member.class,memberId);
//2
Member m2 = jpa.find(Member.class,memberId);

print(m1==m2) //ture를 반환
```
첫번째 조회의 경우, jdbc api를 사용해서 DB server에 접근한다. 그러나 두번째에서는 **캐시**에 존재하는 값을 사용하여 sql을 실행하지 않는다.<br>

2. buffering write<br>
buffer에 모아놓고 트랜잭션 커밋시에 sql을 한번에 처리하게 자동으로 해준다.<br>

3. 지연로딩과 즉시로딩<br>

지연로딩 → 조회를 할 경우, 조회하고 싶은 컬럼이 a,b중에서 a만 있을 때 a만 조회하고 이후에 a와 연관된 b를 사용하고 싶다면 그때 (필요할때) `get` method를 통해서 사용하는 방식으로 `Lazy` 방법. ⇒ 필요할 때만 사용하는 방식인 지연로딩을 자동으로 처리

즉시로딩 → 한번에 여러 테이블이 필요한 경우, 하나의 쿼리 (sql을 한번에 작성)해서 여러 테이블에 바로 접근할 수있도록함.

4. 데이터 접근 추상화<br>
5. 패러다임 불일치 해결<br>
위의 sql을 사용했을 때의 문제점을 해결해준다.<br>

저장, 조회시 persist(), find()를 사용해서 table여러개에 접근하거나 select query를 여러번 사용하는 등 하지 않아도된다.<br>
연관관계 저장시에 객체를 통해서 복잡한 과정없이 `set` method로 연관관계를 저장할 수 있다.<br>
entity를 객체로 접근하면 sql 신뢰성 문제를 해결이 가능하다.<br>
동일한 트랜잭션에서 조회한 엔티티 값이 같음을 보장한다.<br>

----

## JPA를 적용해보자

### Entity를 작성해 객체와 테이블을 매핑한다.

```
@Data
@Entity
public class Item {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

		@Column(name="item_name", length = 10)//없어도됨.
    private String itemName;
    private Integer price;
    private Integer quantity;

    public Item() {
    }

    public Item(String itemName, Integer price, Integer quantity) {
        this.itemName = itemName;
        this.price = price;
        this.quantity = quantity;
    }
}
```

`@Entity` 를 통해서 JPA가 사용하는 객체라고 알려줘서 **객체와 테이블을 매핑하겠다고 알려준다.**

`@Id` 를 통한 pk설정, `@GeneratedValue` 를 통해서 id의 전략(pk를 auto increment)를 설정

`@Table( name = "item")` 으로 작성해도 되지만 class명과 동일하면 생략해도 된다. → 현재는 생략된 상태

`@Column(name="item_name")` 을통해서 객체는 itemName이여서 객체의 필드를 테이블의 컬럼과 매핑한다. → 생략하면 객체 필드의 카멜케이스를 테이블의 컬럼의 언더스코어로 자동으로 변환해서 매핑해준다. (`private Integer price;` 에서는 생략됨을 확인가능. quantity도 마찬가지)

- public() or protected의 기본생성자가 필수.

`public Item() {}` 현재 코드에서는 , public 기본생성자를 빼면 안 됨.

### crud는 어떻게 하면 될까?

repository를 분석해보자!<br>

```
@Slf4j
@Repository
@Transactional
public class japItemRepository implements ItemRepository {

    private final EntityManager em;
    public japItemRepository(EntityManager em) {
        this.em = em;
    }

    @Override
    public Item save(Item item) {
        em.persist(item);
        return item;
    }
    @Override
    public void update(Long itemId, ItemUpdateDto updateParam) {
        Item findItem = em.find(Item.class, itemId);
        findItem.setItemName(updateParam.getItemName());
        findItem.setPrice(updateParam.getPrice());
        findItem.setQuantity(updateParam.getQuantity());
    }

    @Override
    public Optional<Item> findById(Long id) {
        Item item = em.find(Item.class, id);
        return Optional.ofNullable(item);
    }

    @Override
    public List<Item> findAll(ItemSearchCond cond) {
        String jpql = "select i from Item i";
        Integer maxPrice = cond.getMaxPrice();
        String itemName = cond.getItemName();
        if (StringUtils.hasText(itemName) || maxPrice != null) {
            jpql += " where";
        }
        boolean andFlag = false;
        if (StringUtils.hasText(itemName)) {
            jpql += " i.itemName like concat('%',:itemName,'%')";
            andFlag = true;
        }
        if (maxPrice != null) {
            if (andFlag) {
                jpql += " and";
            }
            jpql += " i.price <= :maxPrice";
        }
        log.info("jpql={}", jpql);
        TypedQuery<Item> query = em.createQuery(jpql, Item.class);
        if (StringUtils.hasText(itemName)) {
            query.setParameter("itemName", itemName);
        }
        if (maxPrice != null) {
            query.setParameter("maxPrice", maxPrice);
        }
        return query.getResultList();
    }
}
```

- `EntityManager`를 통해서 데이터베이스에 접근이 가능하고 이는 내부에 데이터소스를 가지고 있다.
<br>

- 이번 예제에서는 `@Transactional` 을 repository에서 적용해줬는데 복잡한 비즈니스 로직이 없어서 서비스 계층에서 트랜잭션을 걸지 않았다. 원래는 서비스단에서 해줘야한다.<br>
    
((JPA는 데이터 변경시에 트랜잭션이 필수이다. 따라서 리포지토리에 트랜잭션을 걸어줌))
<br>

- springboot의 역할<br>
    
`EntityManagerFactory` , `JpaTransactionManager` , 데이터 소스 등등 다양한 설정이 필요하지만 springboot는 해당 과정을 모두 자동화한다.

- update query

cache에서 데이터가 조회되기 때문에 안 보임. 이때 데이터를 보고 싶다면? `@Commit`  을 해줘야함.

[영속성](https://velog.io/@neptunes032/JPA-%EC%98%81%EC%86%8D%EC%84%B1-%EC%BB%A8%ED%85%8D%EC%8A%A4%ED%8A%B8%EB%9E%80)에 의해서 관리되는 경우, “변경 감지”를 자동으로 해주는데 이덕분에 단순히 엔티티를 조회해서 데이터를 변경하면 된다.<br>


### config


```
@Configuration
public class JpaConfig {

    private final EntityManager em;
    public JpaConfig(EntityManager em) {
        this.em = em;
    }

    @Bean
    public ItemService itemService() {
        return new ItemServiceV1(itemRepository());
    }

    @Bean
    public ItemRepository itemRepository() {
        return new jpaItemRepository(em);
    }
}
```

### 예외 변환


Entity Manager는 JPA기술이다.<br>
이 자체만 보면 spring과 관계가 없음.<br>

따라서, 엔티티 매니저는 예외가 발생하면 JPA관련 예외를 발생시킨다. 이때, JPA는 `PersistenceException`과 하위 에러를 발생시킨다.  이렇게 된다면 JPA에 종속적이다. 따라서 스프링 예외로 변경시켜야한다.<br>

![image](https://user-images.githubusercontent.com/74058047/222374841-dbb6abdc-d433-44f1-a78c-5b1e742f550e.png)
<br>


그렇다면 JPA 예외를 어떻게 `DataAccessException` 으로 변화할 수 있을까?<br>
    
    답은) `@Repository` 를 사용

![image](https://user-images.githubusercontent.com/74058047/222375162-a04b4b03-dc6d-4f4e-800a-985fed292bc7.png)<br>
![image](https://user-images.githubusercontent.com/74058047/222375257-b17adf91-930e-4270-8fb5-e8a4b769de90.png)<br>

<br>

`@Repository` 

1. 컴포넌트 스캔의 대상
2. 예외변환 AOP 적용 대상이 된다. PersistenceException을 `DataAccessException` 으로 변환한다.




