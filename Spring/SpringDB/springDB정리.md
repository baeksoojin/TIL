# Spring DB 정리

## 데이터 접근 기술

- JdbcTemplate ( SQLTemplate) : sql동적쿼리가 어렵다는 단점이 존재한다.
- MyBatis (SQLTemplate) : JdbcTemplate보다 더 많은 기능을 제공하는 SQL Mapper로 가장 큰 장점은 “동적쿼리” 작성에 있다. 뿐만 아니라 스프링 예외 추상화 기능을 제공해서 sql을 잘못작성한 경우의 예외가 `DataAccessException` 으로 spring 예외로 잘 터지는 것을 확인할 수 있다.
- JPA, Hibernate (ORM)
- 스프링 데이터 JPA (ORM)
- Querydsl (ORM)

### SQLTemplate의 비교

### 동적쿼리란?

> http request에 의해서 넘어올때 data에 따라서 쿼리가 변화할 수 있는데 이를 동적쿼리라고한다.
> 

ex) filter기능

filter로 넘어온 값에 의해서 쿼리의 값이 변경되어야한다.

MemoryItemRepository에서 findall()이라는 method를 통해서 동적쿼리를 처리한다고 해보자.

- JdbcTemplate

<Repository>

```java
private final JdbcTemplate template;
public JdbcTemplateItemRepositoryV1(DataSource c) {
    this.template = new JdbcTemplate(dataSource);
}
```

우선, JdbcTemplate을 사용하기 위해서 dataSource주입을 받아준다. config파일에서 repository에 dataSource를 주입시켜준다.

```java
@Override
  public List<Item> findAll(**ItemSearchCond** cond) {
      String itemName = cond.getItemName();
      Integer maxPrice = cond.getMaxPrice();

			//이름기준으로 바인딩 -> 순서대로 바인딩하는 것은 개발자가 순서를 체크하면서 넘거야함
			SqlParameterSource param = new BeanPropertySqlParameterSource(cond);

			String sql = "select id, item_name, price, quantity from item"; //동적 쿼리
        if (StringUtils.hasText(itemName) || maxPrice != null) {
            sql += " where";
				 }
        boolean andFlag = false;
        List<Object> param = new ArrayList<>();
        if (StringUtils.hasText(itemName)) {
            sql += " item_name like concat('%',?,'%')";
            param.add(itemName);
            andFlag = true;
}
        if (maxPrice != null) {
            if (andFlag) {
                sql += " and";
            }
            sql += " price <= ?";
            param.add(maxPrice);
        }
        log.info("sql={}", sql);
        //return **template.query(sql, itemRowMapper(), param.toArray());
				return template.query(sql, param, itemRowMapper());//name기준 바인딩**
    }

    //private RowMapper<Item> itemRowMapper() {
    //    return (rs, rowNum) -> {
    //        Item item = new Item();
    //        item.setId(rs.getLong("id"));
    //        item.setItemName(rs.getString("item_name"));
    //        item.setPrice(rs.getInt("price"));
    //        item.setQuantity(rs.getInt("quantity"));
    //        return item;
		//				}; 
		}//sql결과를 받아왔을 때 Item class에 값을 넣어주는데, Mapper를 사용해서 값을 매핑해줘야함
		

		//name기준바인딩
		private RowMapper<Item> **itemRowMapper**() {
		**return BeanPropertyRowMapper.newInstance(Item.class); //camel 변환 지원**
		}

```

+)  parameter를 순서대로 바인딩해서 편리함을 충족시킬 수는 있겠지만, request를 통해서 넘어오는 parameter의 순서가 달라지면, Item값이 서로 섞여서 저장됨.

⇒ 해당 문제는 파라미터 순서를 기준으로 바인딩 하기 때문이니까, `NamedParameterJdbcTemplate` 를 사용해서 이름지정 바인딩 방법을 사용한다.

동적쿼리가 아직 불편함.

1. SOL을 여러줄 작성할 경우, 공백과 엔터 등을 상당히 신경써서 작성.
2. 동적쿼리를 진행할 때, 매우 복잡한 JDBC

이것을 XML 지원을 통해서 편리하게 작성할 수 있다.

- MyBatis

<mapper> 

interface를 사용해서 parameter를 넘기고 xml에서 해당하는 값을 매핑해서 처리하고 다시 넘겨줄때 매핑하기 위해서 사용

interface이지만, `@Mapper` 를 보고 spring이 알아서 구현체를 생성해서 스프링빈으로 등록하고 이것을 **repository에서 의존관계 주입을 받아서 사용할 수 있도록함. → datasource를 내부에서 자동 주입.**

```java
@Mapper
public interface ItemMapper {

    void save(Item item);
    void update(@Param("id") Long id, @Param("updateParam") ItemUpdateDto
            updateParam);
    Optional<Item> findById(Long id);
    List<Item> findAll(ItemSearchCond itemSearch);
}
```

<XML>

```java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="hello.itemservice.repository.mybatis.ItemMapper">
    <insert id="save" useGeneratedKeys="true" keyProperty="id">
        insert into item (item_name, price, quantity)
        values (#{itemName}, #{price}, #{quantity})
    </insert>

    <update id="update">
        update item
        set item_name=#{updateParam.itemName},
            price=#{updateParam.price},
            quantity=#{updateParam.quantity}
        where id = #{id}
    </update>

    <select id="findById" resultType="Item">
        select id, item_name, price, quantity
        from item
        where id = #{id}
    </select>

    **<select id="findAll" resultType="Item">
        select id, item_name, price, quantity
        from item
        <where>
            <if test="itemName != null and itemName != ''">
                and item_name like concat('%',#{itemName},'%')
            </if>
            <if test="maxPrice != null">
                and price &lt;= #{maxPrice}
            </if>
        </where>
    </select>**

</mapper>
```

- resultType : 해당하는 class type으로 값이 mapping 되어서 알아서 들어가도록 처리가 가능.
    
    이때, properties에서 pakage defaule경로를 설정이 가능하고 그 하위의 부분만 적으면 됨.
    
    ```java
    mybatis.type-aliases-package=hello.itemservice.domain
    ```
    

hello.itemservice.domain.Item 경로에 생성되어있는 Item class에 sql을 통해서 받아온 값을 알아서 매핑해준다.
    

---

## SQLTemplate 방식의 문제점으로 인해 생겨난 ORM

| 비교 | SqlTemplate | ORM |
| --- | --- | --- |
| 상속 | 슈퍼타입(item_id, name,price,dtype)-서브타입(item_id,artis)에 걸쳐있는 테이블을 가져오고 싶다면 join을 처리해줘야한다. 많아졌을 때가 문제 | 객체 상속 관계의 경우 , 참조를 활용해서 편리하게 조회가 가능하다. |
| 저장
테이블에 맞춰서 모델링되는 객체 → 연관관계 작성시 객체 모델링은 테이블의 정보를 보고 작성된다 | class Member{
String id; Team team; String username;}
insert문은 insert into MEMBER(member_id, team_id, username) values… | 컬렉션을 통해서 관리하기에 list.add(member) or list.get(memberid), member.getTeam() 등으로 편하게 조회하고 set이 가능. |
| 엔티티 신뢰문제
 | 처음실행하는 sql 범위에 의해서 결정
어디까지가 연결되어있는지 알 수 없어서 정의한 DAO를 따라서 올라가야함 | 모든 객체를 자유롭게 탐색가능 |
| 비교 | new로 생성해서 다른 class | java collection(hash)을 통해서 key,value로 접근해서 같은 class |

⇒ collection 방식을 사용하자. 객체를 기반으로 data를 처리하자.

---

### ORM방식 알아보기

### ORM이란?

> ORM이란 Object-relational mapping 방식을 의미하고 객체 중심적으로 데이터를 편라히게 CURD할 수 있게끔 해주는 데이터 접근기술이다.
> 

- 캐싱기능을 통한 성능 최적화도 제공한다.
    - buffer에 한번에 모아놓고 transaction commit시 한번에 DB에 보내는 방법을 사용한다.
    - 지연로딩 기능을 통해서 원하는 컬럼값만 사용하고 다른 컬럼은 필요할때 get method를 통해서 사용한다.
- 패러다임 불일치를 해결하는데, collection을 사용해서 객체를 기반으로 데이터를 처리하도록 하자는 아이디어를 통해서 구현된다고 생각하자.

 

ORM 방식 비교

- **JPA를 사용한 개발**

> JPA를 사용한 개발은 entitymanager를 통해서 class기반으로 orm을 진행한다. entity를 설계하고 해당 class를 사용해서 crud를 진행하면 된다. 이때, EntityManager를 사용해서 curd method를 사용하면 된다.
> 

<entity>

```java
@Data
**@Entity**
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

- @Entity를 통해서 JPA가 사용하는 객체라고 알려주고 객체와 테이블을 매핑하겠다고 알려준다.
- pulbic() or proteced의 기본생성자는 필수이다.

<crud → transaction에서 적용 : Repository에서 적용해보자면>

```java
@**Transactional**
public class japItemRepository implements ItemRepository {}
```

```java
private final EntityManager em;
public japItemRepository(EntityManager em) {
    this.em = em;
}// EntityManager를 통해서 데이터베이스에 접근이 가능하고 이는 내부에 데이터소스를 포함함
```

```java
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

@Override
public void **update**(Long itemId, ItemUpdateDto updateParam) {
    Item findItem = em.find(Item.class, itemId);
    findItem.setItemName(updateParam.getItemName());
    findItem.setPrice(updateParam.getPrice());
    findItem.setQuantity(updateParam.getQuantity());
  }**// CRUD를 진행할때. .persist나 .find 등을 사용해야하지만 update의 경우에는 필요없음**
```

추가로 update의 경우, transactional에 의해서 영속성관리가 되는데 “변경감지”를 알아서 자동으로 해줘서 단순히 엔티티를 조회해 변경하면 된다.

추가로 `Repository` 를 사용함으로써, 엔티티 메니저에서 발생한 예외를 spring 예외로 proxy가 변환해준다.

- **JPA를 편리하게 사용하게끔 도와주는 interface : Spring Data Jpa**

> JpaRepository<entity,entityid>의 틀을 가지는 인터페이스를 상속받은 **interface** Repository를 생성함으로써 사용이 가능하다.
> 

```java
public **interface** ItemRepository extends JpaRepository<Member, Long> {
 }
```

구현체는 spring data JPA가 프록시 기술을 사용해서 구현 클래스를 생성해준다.

proxy를 구현 class로 springbean에 등록.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bc7e3b52-1926-43f8-8a7e-18ac0e36da48/Untitled.png)

**따라서 개발자는 구현없이 interface만 생성해서 curd를 할 수 있다.**

interface는 예외변환까지 다 제공해준다.

- 쿼리 메서드

```java
public List<Item> findAll(ItemSearchCond cond) {
        String jpql = "select i from Item i";
        Integer maxPrice = cond.getMaxPrice();
        String itemName = cond.getItemName();

//
}
```

위의 코드는 JPA를 사용했을 때 파라미터를 가져와서 sql을 처리하기 위한 시작부분이다. 

메서드 이름으로 쿼리를 자동 생성

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
        **List<Member> findByUsernameAndAgeGreaterThan(String username, int age);**
}
```

- method 이름 규칙

조회: find...By , read...By , query...By , get...By

등등… → 공식문서를 참고하며 개발하면 된다.

- 사용방법

```java
public interface **SpringDataJpaItemRepository** extends JpaRepository<Item,Long> {

//다른기능들은 생략해도 사용가능한 기능들임

    List<Item> findByItemNameLike(String itemName);
    List<Item> findByPriceLessThanEqual(Integer price);
    //쿼리 메서드 (아래 메서드와 같은 기능 수행)
    List<Item> findByItemNameLikeAndPriceLessThanEqual(String itemName, Integer
            price);
//쿼리 직접 실행
    @Query("select i from Item i where i.itemName like :itemName and i.price <= :price")
            List<Item> findItems(@Param("itemName") String itemName, @Param("price")
            Integer price);

}
```

기본기능(생략해도 제공해줌)과 부가기능을 처리한 interface를 repository에서 사용하면 된다.

다만 아직도 query는 문자이기때문에 버그 발생의 문제점을 내포하고 있다.

- **Querydsl**

> query를 java로 작성하여 개발할 수 있도록 지원하는 프레임워크로 컴파일 에러가 나도록해서 에러체크를 쉽게하도록한다.
> 

```java
List<Item> result = query
          .select(item)
          .from(item)
          .where(likeItemName(itemName), maxPrice(maxPrice))
          .fetch();
```

다음과 같이 java code로 query를 구성하는 방식을 의미한다.

- 컴파일 시점에서 오류를 막는다
- 메서드 추출로 코드를 재사용할 수 있다.

---

## 데이터 적용 기술 활용방안

### tradeoff

DI,OCP를 지키기 위해서 어댑터를 도입하고 더 많은 코드를 유지해야한다.

어댑터를 제거하고 구조를 단순하게 가져가지만, DI, OCP를 포기하고 구현한다.

**구조의 안정성 vs 단순한 구조와 개발의 편리성**

- 구현체를 바꿔야할 필요가 있는지가 매우 중요하다. 추상화 비용이 유지보수 관점에서 발생한다는 것을 고려해야한다.
    
    구조의 안정성이 매우 중요한경우에는 추상화 개념을 도입시켜야할 것이다.
    
- 하지만. 그렇지 않다면 그냥 단순한 구조를 가져가는 것도 좋다. 어서플 추상화는 독이 될 수 있다.

### 어떤 것을 사용해야하나?

- 매우 복잡한 통계 쿼리를 주로 작성해야하는 경우에는 ORM은 맞지 않을 수 있기에 MyBatis를 사용한다.
- 실무에서는 보통 JPA, spring data JPA< Querydsl을 함께 95%정도 사용한다. 나머지 5%의 경우에는 순수 sql을 사용해야하는 경우가 존재할 수 있어서 이 경우에는 둘중 하나를 선택한다.
    
    → 이때, 두개가 사용하는 트랜잭션 manager가 다른데, 사실 JpaTransactionManager는 DataSourceTransactionManager의 기능을 대부분 지원해주기 때문에 함께 사용이 가능하다.
    

---

## spring transaction의 의해

### 프록시 내부 호출

프록시를 내부에서 서로 target끼리 내부에서 프록시를 거치지 않고  호출하는 경우

⇒ class를 별도로 쪼개야한다. 


### 초기화

@PostConstruct : AOP가 동작하기 전에 동작해서 transaction이 적용되지 않는다.

### 예외처리

- unchecked, checked 예외 발생시, 그 예외와 하위 예외가 발생하면 트랜잭션을 “rollbak”
- 정상 응답시, “commit”
- 스프링의 예외처리
    
    다만, 스프링은 check예외는 체크 예외는 **커밋**(비즈니스 의미 → 잔액부족의 경우 → 대기상태로)
    
    unchecked exception : 스프링은 언체크는 보통 복구 불가능한 예외인 runtimeerror와 같은 것으로 가정한다. 따라서 **롤백한다**.
    

## 전파

**required**가 기본옵션

> transacion propagaing은 외부 트랜잭션이 수행중일때, 내부 트랜잭션이 추가로 수행되는 경우를 의미한다.
 
이때 묶여서 하나의 트랜잭션으로 처리된다.

- 각각을 논리 트랜잭션. 하나의 묶음을 물리 트랜잭션.
- 모든 논리 트랜잭션이 커밋되어야 물리 트랜잭션이 커밋된다. → 외부,내부 모두 커밋되어야 전체가 커밋된다.
- 하나의 논리 트랜잭션이 롤백되면 물리 트랜잭션역시 롤백된다. → 하나라도 잘못되면 전체가 롤백된다.

논리트랜잭션 2개를 하나로 묶을 수 있는 이유는?

- 외부의 경우, 신규 트랜잭션으로 등록되지만, 내부의 경우 신규 트랜잭션이 아니다. 따라서 **트랜잭션 동기화를 같은 것을 사용**하기 때문에 하나의 물리 트랜잭션으로 묶여서 처리가 가능한 것이다.

- 내부의 롤백

전체가 다 롤백되어야한다. 내부는 신규 트랜잭션이 아니여서 **`rollbackOnly**=true`로 marking만 가능하다. 따라서 트랜잭션 동기화 매니저는 외부 트랜잭션이 커밋되어도 rollback을 진행한다