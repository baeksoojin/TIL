# Autowired의 특징

## type으로 조회하여 DI 처리

- Spring Container 설정의 절차

1. component scan으로 spring container(ac)에 bean이 등록된다.<br>
2. @Autowired를 보고 생성자 주입을 진행한다.<br>

두번째 단계에서 @Autowired를 통해서 method를 보고 class에 의존관계를 주입한다. 이때 1번과정에서 등록된 bean을 찾아서 주입시켜주게 된다.<br>

- type으로 조회
spring bean을 `getBean(discountPolicy.class)`로 조회하는 것처럼 type을 통해서 자동으로 조회한다.<br>
getBean과 동일하게 하위타입이 2가지 이상 있을 때 당연히 오류가 난다.<br>

`expected single matching bean but found 2`와 같은 에러를 뱉는다.

## type이 같을 때의 해결방법

### 여러개가 매칭 되었을 때 파라미터 이름으로 빈 이름을 추가 매칭한다.

`@Autowired private DiscountPolicy discountPolicy` 에서 bean but found 2 에러가 발생했다면? 
=>  `@Autowired private DiscountPolicy rateDiscountPolicy`
<br>
당연히 rateDiscountPolicy라는 이름을 가진 bean 이름이 존재해야할 것이다.<br>

### @Quilifier를 사용한다. (추가 구분자)

빈 이름 자체를 변경하는 것이 아니라 추가적인 방법을 제공하는 것이다.<br>

```
@Component
@Qualifier("fixDiscountPolicy")
public class FixDiscountPolicy implements DiscountPolicy{
```
과
```
@Component
@Qualifier("mainDiscountPolicy")
public class RateDiscountPolicy implements DiscountPolicy{
```
처럼 나타내어 Qaulifier를 통해서 구분해준다<br>
이후 아래의 코드처럼 사용이 가능하다.
```
public OrderServiceImpl(MemberRepository memberRepository, @Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
}
```

- annotation을 직접 만들어서 컴파일 관리하기
이때, 문자는 컴파일 오류에 잡히지 않아서 실수로 이름을 잘못적었을 경우, 어디서 오류가 났는지 알 수가 없음. <br>
컴파일 타입에서 오류가 날 수 있도록 annotation을 생성한다.<br>

~~~
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
@Qualifier("mainDiscountPolicy")
public @interface MainDiscountPolicy {
}

~~~
1~4번까지의 annotation은 @Qualifier에 붙어있는 anntation들인데 복사해서 붙여넣으면 된다.<br>
그리고 이름을 붙이기 위해서 @Qualifier("name")
이렇게 사용하면 된다.<br>
이때 mainDiscountPolicy라는 string은 다른데서 불러서 사용하지 않고 annotation이름인 MainDiscountPolicy를 사용하는 것이기에 annotation이름이 잘못되었다면 컴파일 단위로 오류처리가된다는 장점이 있다.<br>

~~~
@Component
@MainDiscountPolicy //compile error check 위해서 직접 annotation 생성
public class RateDiscountPolicy implements DiscountPolicy{}
~~~

### @Primary

```
@Component
@Primary
public class RateDiscountPolicy implements DiscountPolicy{
```
처럼 우선으로 사용할 Bean에만 primary를 지정해서 명시적으로 획득하는 방법을 사용할 수 있다.<br>

코드가 더 깔끔해지고 한번의 annotation을 사용할 수 있다.<br>

### 우선순위

primary 보다는 qaulifier가 더 상세히 적힌 것이기에 우선순위가 더 높다.<br>

### 조회한 빈이 모두 필요할 때 

**Map, List**를 사용할 수 있다.<br>
```
 static class DiscountService {
        private final Map<String, DiscountPolicy> policyMap;
        private final List<DiscountPolicy> policies;

        @Autowired
        public DiscountService(Map<String, DiscountPolicy> policyMap, List<DiscountPolicy> policies){
            this.policyMap = policyMap;
            this.policies = policies;

            System.out.println("policeMap"+policyMap);
            System.out.println("policies"+policies);
        }


        public int discount(Member member, int price, String discountCode) {
            DiscountPolicy discountPolicy = policyMap.get(discountCode);
            return discountPolicy.discount(member, price);
        }
    }
```

service의 코드를 다음과 같이 바꿀 수 있다.<br>
Map,List를 통해서 여러개의 구현체를 받아서 선택적으로 사용할 수 있다.<br>
**동적으로 Bean을 변경하며 사용해야할 때** 사용하면 다형성을 유지하면서도 편리하게 사용이 가능하다.<br>

