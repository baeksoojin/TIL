# AppConfig를 통한 객체지향적 설계

## Config file

> config파일로 application 개발 중 변경이 가능한 부분에 대해 해당 부분에 어떤 값을 넣어서 사용할지 정의해주는 파일이다.

---

## 왜 사용하는가 [why]


구성영역과 사용영역을 나눠서 SOLID를 적용하며 agile한 방법론을 적용하며 개발을 진행할 수 있기 위해서 사용한다.

**이해돕기!**
만약에 쇼핑몰을 만드는 개발을 진행하다가 기존의 기획에 맞춰서 모두 코드를 작성해놨는데, 기획자의 요구사항 변경으로 인해서 할인정책을 고정할인 가격 정책에서 가격의 10%를 할인해주는 정책으로 변경해야하는 일이 발생했다.<br>
그렇다면 관련된 코드를 모두 변경해야하고 유지 보수가 어렵고 코드가 꼬이는 일이 발생할 수 있다.<br>
따라서 개발을 하기 전 "변경 될 가능성이 있는가?"를 살펴서 개발을 진행해야한다.<br>

객체 지향 설계에 맞추서 역할과 행위의 주체를 구분하여 "interface와 그 구현체를 작성하는 방식으로 개발을 진행한다." 이렇게만 하면 될까?<br>

추상화와 구현체로 분리해놓은 상태라고 가정하고 아래의 service단 코드를 보자.
```

 private final DiscountPolicy discountPolicy = new FixDiscountPolicy();

```
라고 작성을 한다면 SOLID원칙을 과연 모두 지킬 수 있을까?<br>

*그렇지 않다*<br>

위의 코드에서는 interface와 구현체를 분리시켜서 *다형성*을 가져왔지만 이를 적용할 때의 문제점이 발생하여 DIP, OCP의 원칙을 지키지 않고 있다.<br>

- DIP 원칙과 OCP원칙

위의 코드에서 만약 할인정책을 RateDiscountPolicy 구현체로 변경하고 싶다면 어떻게 해야할까?<br>
~~~
 private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
~~~
다음과 같이 구현체를 client에게 제공하는 service단에서 할인정책에 대한 구현체를 수정하는 작업이 필요하다.<br>
따라서 이것은 추상화에도 의존하며 구체화에도 의존하고 있다. 따라서 DIP를 의존하고 있다. 또한 여기서는 확장에는 열려있으나 변경에는 닫혀있어야하지만 구현체의 변경이 이루어지고 있다. 따라서 OCP역시 위반하고 있다. <br>
추가로 service단에서는 역할을 하는 구체적인 구현체를 "지정"해주는 역할을 하면 **많은 역할**을 처리하게 되기에 SRP역시 위반된다.<br>

이를 해결하기 위해서 AppConfig file로 구성파일을 만들어 구현체를 지정하는 파일을 따로 만들어 확장은 열지만 변경을 닫아준다. 또한 추상화에만 의존하도록 해준다. 이렇게 된다면 자연스럽게 SRP을 위반하지 않게 된다.

---

## 어떻게 사용하는가? [How]

config파일은 변경이 가능한 구현체 등을 지정하는데 사용한다.

우선 OrderServiceImpl에서의 위의 코드를 다음과 같이 변경한다.
~~~

private final MemberRepository memberRepository; //interface에만 의존하고 구체화에는 의존하지 않도록 DIP 원칙을 가져감
private final DiscountPolicy discountPolicy; //interface에만 의존하고 구체화에는 의존하지 않도록 DIP 원칙을 가져감

public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
    this.memberRepository = memberRepository;
    this.discountPolicy = discountPolicy;
}
~~~
constructor를 통해서 넘어온 구현체를 service의 discountPolicy interface에 주입이 가능하다. 해당 주입을 해주는 코드를 AppConfig파일에 작성하면 된다.

~~~

public class AppConfig {

    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }//생성자 주입

    private MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }//생성자주입

    public DiscountPolicy discountPolicy(){
        return new FixDiscountPolicy();
    }

}

~~~

구현체를 설정하는 method와 Service에서 사용할 구현체를 interface에 주입시켜주는 역할을 하는 method를 **분리**해서 한눈에 보기 쉽게 작성해야한다.<br>



