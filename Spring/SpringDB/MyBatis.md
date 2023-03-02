# MyBatis

기존의 jdbcTemplate에서 동적쿼리를 처리하고 이때, 필터가 많아지면 코드가 계속 늘어나야하는데 이를 처리하기 어렵다는 단점이 존재했다.<br>

MyBatis는 이러한 문제점을 해결해준다.<br>

> [여기](https://mybatis.org/mybatis-3/)에서는 MyBatis is a first class persistence framework with support for custom SQL, stored procedures and advanced mappings로 MyBatis를 정의하고 있다.

동시에 대부분의 JDBC code와 수동으로 처리해야하는 파라미터를 줄일 수 있다고 나온다.<br>
MyBatis는 Java POJO와 Map interface를 구성하는데 XML 혹은 Annotation을 사용하여 디비를 나타낼 수 있다.<br>

-----


## MyBatis란

JDBCTemplate보다 더 많은 기능을 제공하는 SQL Mapper이다.<br>

**장점**<br>
- JDBCTemplate에서의 쿼리는 string으로 이어서 작성할 경우 공백과 엔터를 상당히 신경써서 작성해야하지만, MyBatis는 문서처럼 작성하는 것이기에 신경쓰지 않아도 된다.<br>
- 동적쿼리를 진행할때, 매우 복잡한 JDBC를 극복하고 XML을 지원하여 편리하게 작성이 가능하다.<br>

다만, MyBatis를 사용하기 위해서는 gradle에 등록이 필요하다. 이때, spring이 버전 관리를 해주지 않기에 버전을 함께 명시해줘야한다.<br>

------


## MyBatis 사용하기

### config

- gradle 등록<br>
`implementation 'org.mybatis.spring.boot:mybatis-spring-boot-starter:2.2.0'`<br>
- properties 설정<br>
```

mybatis.type-aliases-package=hello.itemservice.domain
mybatis.configuration.map-underscore-to-camel-case=true
logging.level.hello.itemservice.repository.mybatis=trace
```
MyBatis는 package name type 정보를 사용할 때 적어줘야하는데, 매번 경로를 적어주는 것은 번거롭기에 package를 명시하여 패키지 이름을 생략한다.<br>
sql의 언더스코어 표기방법을 java 객체의 camel표기법으로 자동변환을 하도록 세팅한다.<br>
sql logging을 할 수 있도록 'trace'로 logging을 진행하도록 설정한다.<br>

### Mapper 생성과 XML mapping

xml을 활용해서 동적 쿼리를 편하게 작성하는데 Mapper interface를 사용하여 mapping XML을 호출해주는 매퍼 인텊이스를 생성해야한다.<br>

- mapper

```
@Mapper
public interface ItemMapper {

    void save(Item item);
    void update(@Param("id") Long id, @Param("updateParam") ItemUpdateDto
            updateParam);
    Optional<Item> findById(Long id);
    List<Item> findAll(ItemSearchCond itemSearch);
}
```
`@Mapper` annotation => interface를 생성하면 자동으로 스프링빈에 등록해서 이것을 repository에서 의존관계를 주입받아서 사용할 수 있도록한다.<br>
interface의 method의 이름 => xml mapper에서 code의 `name`과 매핑된다.<br>
`@Param` => parameter가 두개이상 사용되는 경우 반드시 해당 어노테이션을 통해서 구분해준다.<br>

- xml code

```
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

    <select id="findAll" resultType="Item">
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
    </select>

</mapper>
```


**query** => xml의 id에 method 이름을 적어주고 tag를 insert, select, update등등의 쿼리로 적어준다.<br><br>
resultType => 반환 타입을 명시해야하는데 사실상 모든 패키지명을 다 적어줘야하지만 properties에서 명시한 패키지 경로가 디폴드로 적용되어 다 적어주지 않아도 된다.<br> `<select id="findAll" resultType="Item">` <br>
이라면 query 결과를 바로 Item 객체에 저장한다. <br><br>
parameter => `#{}`을 통해서 매개변수를 바인딩한다. <br> `@Param("id") => #{id}`<br>
만약에 `${}`를 사용한다면 문자 그대로를 처리하기 때문에 "SQL 인젝션 공격에 당할 수 있는데 주의해야한다."<br><br>
**동적 쿼리** => MyBatis는 사실상 동적 쿼리를 편하게 작성하기 위해서 사용하는 것이기에 가장 중요하다.<br> MyBatis에서는 동적 쿼리를 편하게 작성할 수 있도록 여러 문법들을 지원하는데 그 중에서 `where`, `if`를 사용했다.<br>

```
<where>
    <if test="itemName != null and itemName != ''">
        and item_name like concat('%',#{itemName},'%')
    </if>
    <if test="maxPrice != null">
        and price &lt;= #{maxPrice}
    </if>
</where>
```

`<where>`가 아닌 where로 적는다면 어떻게 될까?<br>
**if문이 하나라도 걸리지 않는다면, select ~ from ~ where** 과 같이 where뒤에 조건문이 걸리지 않아서 "에러"가 발생한다.<br>
이것을 보완하고 대비하기 위해서는 `<where>`와  `<if>`를 함께 적어준다. 이때, **if문이 하나라도 통과되면 where뒤에 and는 사라지고 그 뒤의 구문만 적용될 것이다.** <br>

### 실행을 위한 Config file 작성

```
@Configuration
@RequiredArgsConstructor
public class MyBatisConfig {

    private final ItemMapper itemMapper;

    @Bean
    public ItemService itemService() {
        return new ItemServiceV1(itemRepository());
    }
    @Bean
    public ItemRepository itemRepository() {
        return new MyBatisItemRepository(itemMapper);
    }


}
```

**코드 해석**<br><br>

구성정보에 transaction을 위한 datasource가 없어서 의문이 들것이다.
MyBatis module이 datasource를 읽어와서 transaction과 연결을 알아서 시켜준다.<br>
따라서 transaction이 자동으로 처리되기에 고려하지 않아도된다.<br><br>
**DI**<br>
service에서 Mapper interface인 `itemMapper`를 repository에 주입받아서 repository를 service에 주입한다.<br>

## MyBatis 원리 이해하기

그렇다면 어떻게 이렇게 사용할 수 있었던 것일지 알아보자.<br>

### interface를 구현체 정의 없이 사용하였다.[HOW?]

결론 : 구현체 정의만 직접 하지 않았을 뿐 프록시를 통해서 알아서 구현체를 생성해놓기에 구현체는 존재한다.<br><br>

과정 : MyBatis의 스프링 연동 모듈이 매퍼를 보고 interface의 구현체를 동적 프록시 기술을 사용해서 생성하는데 이렇게 구현체를 스프링 빈으로 등록하고 이것을 사용하게 된다.<br>

ex) 프롤기 구현체를 사용하는 것을 살펴보자!<br>
save method에 아래의 로그를 찍어보면, 어떻게 어떻게 나올까?
```

log.info("itemMapper class = {}",itemMapper.getClass());

```
![image](https://user-images.githubusercontent.com/74058047/222357955-c12512fe-7aba-4bf6-8454-0d3bd4c337a2.png)

<br>
다음과 같이 proxy 객체인것을 확인이 가능하다.<br>

------

MyBatis는 native query를 사요알 경우, jdbcTemplate과 둘중에서 고려하여서 사용하면 된다.<br>
실제로는 ORM을 사용하여 sql문을 작성하지 않고 **"객체를 관계형 디비에 저장하는" 방식인** ORM이 주로 사용되는데 JPA를 사용한다.<br>

이때, jpa의 경우 spring data jpa, querydsl을 통해서 더 편리하게 사용이 가능한데 반드시 알아야한다.<br>