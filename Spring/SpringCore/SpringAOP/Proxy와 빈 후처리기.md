# Proxy와 빈 후처리기

## BeanPostProcessor

- Bean

@Bean annotation을 사용해서 직접 스프링빈으로 등록하거나 컴포넌트 스캔으로 스프링빈을 등록하게 된다.

스프링빈으로 등록이 되어야 스프링 컨테이너에서 관리되고 이후는 스프링빈을 spring container에서 조회해서 사용하게 된다.

proxy를 등록할 때는 Spring Bean으로 직접 config file을 작성해서 복잡한 구성단계 과정을 거쳐야 spring bean으로 등록되기 전에 proxy를 반환하는 proxy Bean으로 변경이 가능하다. 그렇다면, 우리는 자동 component scan 방법을 통한 Bean등록 방식으로는 Proxy를 사용할 수 없게된다.

이러한 단점을 극복하기 위해서 존재하는 것이 빈 후처리기이다.

빈 후처리기

> Spring Bean이 먼저 등록되어버리면 proxy를 설정하지 못하기 때문에, 빈이 스캔된 후 등록되기 직전에 조작하고 싶은 무언가가있다면 빈 후처리기인 `BeanPostProcessor`를 사용한다. 이를 번역하면 빈 후처리기가 된다.
> 

## BeanPostProcessor의 Process

![image](https://user-images.githubusercontent.com/74058047/224775507-80c3d5a1-2bfc-4b7c-b61a-9792bc3fdbef.png)

- 생성 : 스프링빈 대상이 되는 객체를 정의 (component scan 포함)
- 후처리기에 전달 : 스캔된 객체를 BeanPostProcessor에 전달
- 후 처리 작업 : 처리
- 등록 : 반환된 Bean을 Container에 등록

proxy 적용시에 BeanPostProcessor를 사용하면 된다.

빈 후처리기를 proxy 등로을 위해서 사용한다면 그 범위를 제한해주기 위해서 패키지를 기준으로 프록시 적용범위를 나눠줄 수 있지만, 포인트 컷을 사용하면 더 깔끔할 것이다.

참고로, advisor의 경우 포인트컷을 가지고 있다. 이때 스프링 AOP는 포인트컷을 사용해 프록시 적용 대상 여부를 체크하는데 Spring AOP PointCut의 경우 나중에 자세히 알아보도록하자!

proxy를 사용하기 위해서 BeanPostProcessor를 사용할때의 장점은 프록시를 생성하는 부분을 하나의 BeanPostProcessor를 등록하는 과정에서 한번에 처리가 가능하며 컴포넌트 스캔 자동 빈 등록 과정에서도 역시 프록시 적용이 가능하다는 것이다.

그래서 BeanPostProcessor를 사용해서 proxy를 처리하면 된다.

그러나, **스프링은 이미 proxy 생성을 위한 BeanPostProcessor를 제공하기에 이를 사용하면 된다.** 

---

## Spring이 제공하는 빈 후처리기 AutoProxyCreator

- config (build.gradle)

`org.springframework.boot:spring-boot-starter-aop` 를 build.gradle에 추가해야한다.

aspectj 관련 라이브러리를 등록하고 AOP 관련 클래스를 자동으로 스프링 빈에 등록한다.

![image](https://user-images.githubusercontent.com/74058047/224775607-e261a038-98b2-45e7-8ee5-499322c48b81.png)

- `AutoProxyCreator`

위의 설정을 통해서 `AnnotationAwareAspectJAutoProxyCreator` 라는 빈 후처리기가 스프링 빈에 등록되는데 프록시를 자동으로 생성해준다.

`Advisor` 를 찾아서 프록시가 필요한 곳에 자동으로 프록시를 적용해준다.

- 참고로 Advisor는 pointcut(proxy 적용 범위설정) + Advise(proxy구현)로 구성이 되어있다.

AutoProxyCreator 상속관계를 따라서 올라가보면 상단에 `BeanPostProcessor` 가 존재한다.

---

## @Aspect AOP

`AnnotationAwareAspectJAutoProxyCreator` 를 통해서 `Advisor` 등록을 통해서 자동 프록시 생성 및 적용범위 제한 등을 제공할 Proxy 후처리기 기능을 사용하는 대신에, `@Aspect` annotation을 활용할 수 있다.

단순히 말하면, 코드로 작성했던, Advisor 부분을 anootation으로 사용하는 것이다.

- @Aspect : proxy를 적용할 class에 적용
- @Aroud : 하위 method에 advise를 정의하고 argument로 pointcut을 넘기기
    - ProceedingJoinPoint를 사용해서 Signature를 불러와서 어떤 메서드에서 동작하는지를 알아볼 수 있음
    - ProceedingJoinPoint를 사용해서 logic을 `.proceed()`를 통해서 자동으로 타겟의 로직을 처리할 수 있음.