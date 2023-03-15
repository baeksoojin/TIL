# 스프링 AOP

---

스프링 AOP를 알아보기 전에 스프링 AOP에 대해서 알아보자!

## AOP란 무엇인가

**AOP를 사용하지 않았을 때의 문제점)**

핵심로직과 부가 기능이 존재하는데, 두개가 하나의 로직을 완성하게 된다.

보통 부가 기능은 여러 클래스에 걸쳐서 사용하게 된다. 예를 들어서 로깅을 해야하는 요구사항에서는 여러 곳에서 사용될 것이다. 따라서 이러한 부가 기능을 cross-cutting concerns라고한다. 

다만, 부가 기능을 여러 곳에 각각 적용하려면 번거로울 뿐만 아니라 수정시에, 모두 찾아가면서 수정해야할 필요가 있다.

부가 기능을 적용할 때 아주 많은 반복, 중복 코드가 발생이 가능하다.

심지어 수정을 할 때 직접 찾아가면서 많은 수정이 필요하다.

**문제점을 해결하기 위해 Aspect를 도입해서 해결한다)**

그렇다면, 핵심기능에서 부가기능을 분리해서 한곳에서 관리하면 위의 문제를 해결할 수 있을 것이다.

> 부가 기능과 부가 기능을 어디에 적용할지( advisor, pointcut )의 기능을 합해 하나의 모듈로 만든 것을 aspect라고 하고 aspect를 사용한 프로그래밍 방식을 관점 지향 프로그래밍이라고해서 Aspect-Oriented Programming 즉, AOP라고한다.
> 

그 동작은, 횡단 관심사를 모듈화하여 핵심로직에서 사용하는 것을 의미한다.

---

## AOP의 장점

AOP 구현으로 AspectJ를 많이 사용한다. 이는 자바 프로그래밍 언어에 대해서 관점 지향성을 확장 시켜주는데, 즉, 옆으로 퍼져서 여러 핵심기능에 합쳐져있던 cross-cutting concerns를 **모듈화해서** 처리한다.

횡단 관심사는 AspectJ를 사용해, 오류 검사 및 처리, 동기화, 성능 최적화(캐싱기능), 모니터링 및 로깅 처리에 대한 깔끔한 모듈화가 가 가능하다.

---

## AOP 적용방식

- Weaving
    
    Aspect와 실제 코드를 연결해서 붙이는 작업을 하는 것을 위빙이라고한다.
    
    Weaving은 컴파일 시점, 클래스 로딩 시점, 런타임 시점에 적용될 수 있다.
    
    - 컴파일 시점의 경우, 클래스 파일을 만드는 시점에 부가기능이 붙어서 들어가는데 이때, 해당 클래스가 부가 기능에 대한 적용 대상인지 먼저 확인하는 작업이 필요한데 AspectJ Compiler가 따로 필요하다. 따라서 특별한 컴파일러를 필요로하고 복잡하다는 단점이 존재한다.
    - 클래스 로딩 시점의 경우, 클래스를 JVM 내부의 클래스 loader에 보관하고 중간에서 class file을 조작한 다음 JVM에 올린다. 자바 언어는 class file을 JVM에 저장하기 전에 조작할 수 있게끔 기능을 제공하는데 참고로 수많은 모니터링 툴들이 해당 방식을 사용한다. 클래스 로딩 시점에서 Aspect를 로드타임 위빙이라고한다. 다만, 클래스 로더 조작기를 지정해야하는데 이부분이 번거롭고 운영하기 어렵다는 단점이 존재한다.
    - 런타임 시점의 경우, 프록시 방식을 사용해서 처리하는 것이다. 런타임에서 프록시 방식을 사용하기 때문에 복잡한 설정과 옵션 추가를 하지 않아도 된다는 장점이있다. 다만, 프록시를 사용하기에 일부 기능상 제약이 존재할 수 있다. 그러나 자바 언어가 제공하는 범위 안에서 부가 기능을 적용하고 스프링 컨테이너의 도움을 받고 DI, BeanPostProcessor만을 통해서 제공하기에 다른 것에 의존하지 않는다는 장점이 존재한다.
    
    스프링 AOP는 런타임시점에서의 Weaving을 사용한다.
    
- JoinPoint와 Spring
    
    > JoinPoint란 부가기능 적용 가능한 지점을 의미한다.
    > 
    
    원래의 AspectJ의 JoinPoint의 경우에는 바이트코드 조작을 사용해 생성자, 필드, static, method 등등 모든 영역에서 부가기능 로직을 적용할 수 있다.
    
    그러나, proxy를 사용하는 Spring AOP는 “메서드”실행 시점에서만 AOP를 적용할 수 있다.
    
    따라서, 생성자나 static method, field값 접근에 프록시 개념이 적용될 수 없다.
    
    Spring AOP는 Spring Bean에 등록해야 AOP가 적용될 수 있다는 것을 기억해야한다.
    
    예를 들어서, 주문 로직에서 OrderService, OrderRepository 등등을 생성했고 메서드들에 로그처리 부가기능을 넣을 것이라고면 OrderService, OrderRepository 의 부가기능을 적용할 메서드들이 JoinPoint가 된다. 
    

위에서 알아본것처럼 AOP의 경우 ApectJ를 사용해도 되고 Proxy를 사용해서 Spring AOP를 사용해도 된다. 

스프링 AOP는 별도의 추가 자바 설정없이 스프링만 있으면 AOP를 사용할 수 있기에 AsepctJ를 사용하지 않고 사실 스프링이 제공하는 AOP만 사용해도 대부분의 문제를 해결할 수 있다.

따라서, AspectJ 자체를 사용하기보다는 Spring AOP에 대해서 알아보자!

---

## Spring AOP

## Advisor

`@Aspect` 를 통해서 프록시를 생성하고 Spring Bean으로 등록한다.

proxy생성)

```java
@Aspect
@Slf4j
public class AspectV1 {

    @Around("execution(* tutorial.aop.order..*(..))")
    public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {
        log.info("[log] {}", joinPoint.getSignature()); //join point 시그니처
        return joinPoint.proceed();
    }

}
```

test)

위의 프록시를 testcode를 통해서 작동시켜보자.

`@Import(AspectV1.class)` 를 통해서 spring Bean으로 등록시키고 진행한다.

```java
@Test
void aopInfo(){
    log.info("isAopProxy, orderService ={}", AopUtils.isAopProxy(orderService));
    log.info("isAopProxy, orderService ={}", AopUtils.isAopProxy(orderRepository));
}
```

![image](https://user-images.githubusercontent.com/74058047/225245581-4ec553e5-dbb1-4a90-9c11-188b7ba23468.png)


## pointcut

`@Pointcut` annotation을 사용해서 포인트컷을 분리할 수 있다.

```java
@Pointcut("execution(* tutorial.aop.order..*(..))") //pointcut expression
private void allOrder(){} //pointcut signature

@Around("allOrder()")
public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {
    log.info("[log] {}", joinPoint.getSignature());
    return joinPoint.proceed();
}
```

- pointcut분리

```java
public class Pointcuts {

    @Pointcut("execution(* tutorial.aop.order..*(..))") //pointcut expression
    public void allOrder(){} //pointcut signature

    @Pointcut("execution(* *..*Service.*(..))")
    public void allService(){}

    @Pointcut("allOrder() && allService()")
    public void orderAndService(){}

}
```

- pointcut 지시자(PCD : Pointcut Designator)

> 포인트컷 표현식에서의 `execution`과 같이 포인트컷을 지시하는 것을 말한다.
> 

여러 지시자가 존재하지만, `execution`을 가장 많이 사용한다.

## 부가기능 로직 순서 보장

- `@Order` 를 사용

```java
@Aspect
@Order(2)
public static class Ex1{

    //생략
}

@Aspect
@Order(1)
public static class Ex2{

    //생략
}
```

## Advise

joinpoint를 기준으로 실행이전, 실행 후, 모든 영역 개발 등등으로 어디까지 직접 작성할건지에 대한 영역을 설정할 수 있다.

Annotation

- `Around`
    - 모든 영역 → proceed()를 호출해야 다음 대상이 호출됨
- `Before`
    - joinpoint process 실행이전까지 개발 이후는 알아서 처리
- `After`
    - joinpoint process 실행후만 개발 이전은 알아서 처리 → finally logic이라고 생각하면 됨
    - `AfterReturning` 을 사용하면 조인포인트가 정상 완료됐을 때 처리
    - `AfterThrowing` 을 사용하면 메서드가 예외를 던지는 경우에 해당하는 코드를 작성

좋은 설계를 위해서 제약을 둬야한다.

제약을 두는 annotation을 사용해서 범위를 한정짓고 실수를 미연에 방지해야한다.