# springcontainer와 singleton 보장

## singleton 보장

[web application과 singleton](https://github.com/baeksoojin/TIL/blob/main/Spring/SpringCore/web%EA%B3%BC%20singleton.md) 에서 알 수 있듯이, spring container를 사용하면 코드 작성없이 singleton을 보장해줄 뿐만 아니라 직접 면 코드로 singleton pattern을 맞춰서 작성하며 생길 수 있는 문제점을 해결해준다.<br>

그렇다면 spring에서 singleton 보장이 어떻게 가능한 것일까?를 알아보려고한다.<br>
아래의 코드를 보자.<br>

```

@Configuration
public class AppConfig {

    @Bean
    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }//생성자 주입

    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    @Bean
    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }//생성자주입

    @Bean
    public DiscountPolicy discountPolicy(){
        return new FixDiscountPolicy();
    }


}

```
다음곽 같이 java코드로 작성된 appConfig.class file이 존재한다고 할때, MemberService에서 ` new MemoryMemberRepository() `를 통해 MemoryMemberRepository()를 생성하고 OrderService에서도 ` new MemoryMemberRepository() `를 통해서 또 하나의 MemoryMemberRepository instance를 생성하게 된다.<br>

그렇다면 singleton에 어긋나는 것이 아닌가??<br>


```

public class ConfigurationSingletonTest {

    @Test
    void configurationTest(){
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

        MemberServiceImpl memberService = ac.getBean("memberService", MemberServiceImpl.class);
        OrderServiceImpl orderService = ac.getBean("orderService",OrderServiceImpl.class);
        MemoryMemberRepository memberRepository = ac.getBean("memberRepository", MemoryMemberRepository.class);
        //service 구체타입으로 꺼내면 좋지않지만..singleton test를 위해서 inpl에 만들어놓은 것을 사용하기 위해서 다음곽 같이 적음

        MemberRepository memberRepository1 = memberService.getMemberRepository();
        MemberRepository memberRepository2 = orderService.getMemberRepository();

        System.out.println("memberService -> memberRepository"+ memberRepository1);
        System.out.println("orderService -> memberRepository"+ memberRepository2);
        System.out.println("memberRepository"+ memberRepository);

        Assertions.assertThat(memberRepository1).isSameAs(memberRepository);
        Assertions.assertThat(memberRepository2).isSameAs(memberRepository);

    }

}

```
3번의 new로 인해서 새로운 instance가 생성되어야 할 것 같다.<br>
하지만 결과는<br>
memberService -> memberRepositorytutorial.core.member.MemoryMemberRepository@4d157787<br>
orderService -> memberRepositorytutorial.core.member.MemoryMemberRepository@4d157787<br>
memberRepositorytutorial.core.member.MemoryMemberRepository@4d157787<br>
다음과 같이 나온다.<br>

**어긋나지 않는다. spring을 이용하면 singleton을 보장해준다.**


```

@Configuration
public class AppConfig {

    @Bean
    public MemberService memberService() {
        System.out.println("call AppConfig.memberService");
        return new MemberServiceImpl(memberRepository());
    }//생성자 주입

    @Bean
    public MemberRepository memberRepository() {
        System.out.println("call AppConfig.memberRepository");
        return new MemoryMemberRepository();
    }

    @Bean
    public OrderService orderService() {
        System.out.println("call AppConfig.orderService");
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }//생성자주입

    @Bean
    public DiscountPolicy discountPolicy(){
        return new FixDiscountPolicy();
    }


}
```

log를 찍은다음에 call이 어떻게 이루어지는지 확인해보았다<br>
1. call AppConfig.memberService
2. call AppConfig.memberRepository
3. call AppConfig.orderService
3개만 호출된다.<br>
singleton을 보장하기 위해서 method가 한번만 호출되고 있는 것을 확인할 수 있다.<br>

어떻게 위와같이 method가 한번만 호출되는 것이고 singleton이 보장될 수 있는 것인지 알아봐야할 것이다<br>

----

## 어떻게 singleton이 보장되나

spring container는 AppConfig를 상속받은 임의의 다른 클래스를 만들고 만들어진 다른 클래스를 스프링 빈으로 등록한다.<br>
~~~
   @Test
    void configurationDeep(){
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
        AppConfig bean = ac.getBean(AppConfig.class);

        System.out.println("bean = "+bean.getClass());
    }

~~~

결과는 ```bean = class tutorial.core
AppConfig$$EnhancerBySpringCGLIB$$69863c26``` 이렇게 나오는데, CGLIB라는 바이트코드 조작 라이브러리를 사용해서 만들어진 다른 클래스가 스프링 빈으로 등록된 것임을 확인 할 수 있다<br>

**CGLIB로 생성된 appconfig의 역할**
- 이미 등록되어있는지 체크하여 반환
    호출했을때, 이미 있다면 스프링 컨테이너에서 반환하게 된다.<br>
    호출했을때, 없다면 스프링 컨테이너에 등록하여 반환한다.<br>

위의 과정을 CGLIB code에서 처리하게된다.<br>

appConfig를 name으로 value를 CGLIB를 통해 만들어진 다른 클래스가 container에 등록되어서 싱글톤이 보장되는 것이다<br>

이때 주의할 점이 있다. 그것을 살펴보자!

---

## @Configuration

*@Configuration을 사용하지 않는다면?*<br>
    bean = class tutorial.core.AppConfig 순수한 AppConfig가 나오게된다. 그러면서 MemberRepository가 1번이 아닌 요청한 만큼 호출된다.<br>


- Configuration annotation의 역할
    spring container로 bean을 관리하게끔 한다.<br>
    > 바이트코드를 조작하는 CGLIB 기술을 사용해서 싱글톤을 보장하는 역할을 한다.

요약하자면...<br>
**configuration과 bytecode를 통해서 spring bean에 등록되어있는지 체크하는 과정을 거쳐서 싱글톤을 보장한다**는 것이다

[참고]
- Autowired를 통해서 bean에 등록된 것을 자동으로 주입시켜줄 수 잇는 또 다른 방법도 존재한다.