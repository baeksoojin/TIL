# DI (Dependency Injection)

DI 자체에 대한 개념은 생략한다!!<br>

----

## 다양한 의존관계 주입 방법

spring에서 의존관계 주입 하는 방법은 총 4가지가 존재한다.

1. 생성자 주입
2. setter(수정자) 주입
3. 필드 주입
4. 일반 메서드 주입

### 생성자 주입

> 생성자를 통해서 의존관계를 주입하는 방법이다.

~~~
@Component
public class OrderServiceImpl implements  OrderService{

    private final MemberRepository memberRepository; //interface에만 의존하고 구체화에는 의존하지 않도록 DIP 원칙을 가져감
    private final DiscountPolicy discountPolicy; //interface에만 의존하고 구체화에는 의존하지 않도록 DIP 원칙을 가져감

    @Autowired
    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
}
~~~

코드소개)<br>
spring component scan시에 spring bean에 등록시, 생성자를 호출하는 순간에만 @Autowired를 보고 의존관계를 주입한다.<br>
SpringBean이 Bean 등록을 하기 위해서 생성자를 불러와한다. 그렇기 때문에 Bean 등록 이후 원래는 의존관계 주입이 일어나지만 생성자를 가져올때 @Autowired를 보고 의존관계 주입이 일어난다.<br>
위의 코드에서는 생성자가 1개이기에, @Autowired를 생략해도 된다.<br>

장점)
- 생성자 호출 시점에만 "단 한번만" 호출한다.<br>
- **불변, 필수** 의존관계일 때 사용한다.
    첫 등록 이후 절대 수정할 수 없도록 한다.

### setter(수정자) 주입

> setter를 통해서 주입하는 방법이다.

```
@Component
public class OrderServiceImpl implements  OrderService{

    private  MemberRepository memberRepository; //interface에만 의존하고 구체화에는 의존하지 않도록 DIP 원칙을 가져감
    private  DiscountPolicy discountPolicy; //interface에만 의존하고 구체화에는 의존하지 않도록 DIP 원칙을 가져감

    @Autowired
    public void setMemberRepository(MemberRepository memberRepository){
        this.memberRepository = memberRepository;
    }

    @Autowired
    public void setDiscountPolicy(DiscountPolicy discountPolicy){
        this.discountPolicy = discountPolicy;
    }
}
```

코드설명)<br>
Spring이 Bean을 등록한 이후에 @Autowired를 보고 의존관계 주입을 실행한다. 그때 setter에 적용된 Autowired annotation을 보고 의존관계를 주입한다.<br>
spring은 bean 등록 단계 이후에 @Autowired를 통해서 의존관계를 주입하는 단계로 나눠지기 때문에 spring container의 두번째 단계에서 의존관계 주입이 일어난다<br>

장점)<br>
주입할 때 spring bean에 등록되지 않았을 경우에도 주입이 가능하다. 다만 주입 대상이 없을 때도 동작하게 하려면 `@Autowired(required=false)`로 설쟁해야 오류를 피한다. <br>
변경 가능성이 있는 의존관계에 사용한다<br>

### 필드주입

> 필드에 값을 넣는 방법이다.

```
@Component
public class OrderServiceImpl implements  OrderService{

    @Autowired
    private  MemberRepository memberRepository; //interface에만 의존하고 구체화에는 의존하지 않도록 DIP 원칙을 가져감
    @Autowired
    private  DiscountPolicy discountPolicy; //interface에만 의존하고 구체화에는 의존하지 않도록 DIP 원칙을 가져감

```

코드설명)<br>
필드에 바로 의존성을 주입한다. spring bean 등록 이후 @Autowired를 통해서 필드 코드를 통해서 바로 의존성을 주입한다.<br>

장점)
- 간결하다.
- "springboot를" 활용한 "test"에서 필드주입 방법을 사용해서 test code를 작성할 때 활용하기 쉽다.
    test code는 누가 건드리거나 수정할 일이 없으니까 가능

단점)
- 순수한 "java로 단위테스트를 진행할때", 의존성을 주입할 값을 변경하고 싶을 때 해당 필드로 접근이 불가능해서 제대로된 테스트가 불가능하다.
    값을 넣고 싶은데 접근이 불가능해서 setter를 따로 열어주게 될 수 밖에 없다.<br>
- DI 프레임워크가 없다면 아무것도 할 수 없다.

### 일반 메서드 주입

> 일반 메서드를 통해서 주입 받을 수 있다.

~~~
@Autowired
    public void init(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
~~~
코드설명)<br>
init이라는 아무 method를 통해서 의존관계를 주입시키는 방법으로 setter와 비슷하다.<br>

장점)<br>
- 한번에 여러 의존관계 주입이 가능하다.
특징)<br>
- 일반적으로 잘 사용하지 않는다.

----

## Autowired의 option처리

Autowired annotation의 기본 옵션은 required = True이다. <br>
즉, 주입할 대상이 스프링 빈에 등록되어있지 않은 경우 error가 발생한다<br>
만약 주입할 대상이 없어도 동작하게끔 하기 위해서는 어떻게 해야하나? @Autowired의 option처리에 대해서 알아보자!<br>

### required = false
```
@Autowired(required = false)
public void setNoBean1(Member noBean1) {
    System.out.println("noBean1" + noBean1);
}
```
다음과 같이 testcode에 작성을 한 후에 method가 정의된 class를 container에 넣어준다면? Bean1이 등록되어있지 않지만 error를 반환하지 않는다<br>
` required = false `를 통해서 주입할 대상이 null이라면 해당 method 자체를 호출하지 않는다.<br>
따라서 log값이 터미널에 아예 찍히지 않는다.<br>

### @Nullable
~~~
@Autowired
public void setNoBean2(@Nullable Member noBean2) {
    System.out.println("noBean2 " + noBean2);
}

~~~
@Nullable을 사용한다면 springbean에 등록되어 있지 않아 null 이지만, 호출은 된다<br>
따라서 log값이 "noBean2 null"로 찍힌다<br>

### Optional
```
@Autowired
public void setNoBean3(Optional<Member> noBean3){
    System.out.println("noBean3 " + noBean3);
}
```
java8을 활용해 Optional을 사용한다면 Optional.empty 즉 Bean이 없어도 가능하고 이 역시 호출된다.<br>
따라서 log값이 "noBean3 Optional.empty"으로 찍힌다<br>

------

