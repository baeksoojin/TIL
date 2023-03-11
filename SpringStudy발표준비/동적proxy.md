# 동적 proxy

### **프록시 패턴의 단점**

프록시를 적용할때 로직은 다 비슷한데 중복이 발생함.

ex ) controller, service, repository에서 log를 찍기 위한 proxy class를 다 만들어서 적용해야하는데 이때, 로직은 다 비슷함.

중복발생 → 중복제거를위한 방법은 없나?

실시간으로 프록시를 만들어서 처리 → 동적 프록시를 활용

---

---

겹치는 부분이 존재. 동일한 로직이 아닌 변경가능한 로직을 따로 그것만 넘겨줄 수 있지않나? 사실 중간에 있어서 그렇게 만드는 것은 어려운데 아래의 2가지 방법을 통해서 동적 프록시 기술을 적용시킴으로써 가능하다.

## 동적 프록시 기술

자바가 제공하는 JDK 동적 프록시기술이나 CGLIB를 사용한다.

- JDK 동적 프록시 기술 이해를 위해서 “리플렉션”을 이해해야함.

> 리플렉션 기술이란 클래스나 메서드의 메타정보를 사용해서 동적으로 공통 로직을 처리할 수 있는 기술이다.


<ex>


변경 전

```java
@Slf4j
  public class ReflectionTest {
      
			@Test
      void reflection0() {

			Hello target = new Hello(); //공통 로직1 시작
			log.info("start");

			String result1 = target.callA(); //호출하는 메서드가 다름
			log.info("result={}", result1); //공통 로직1 종료

			//공통 로직2 시작 log.info("start");
			String result2 = target.callB(); //호출하는 메서드가 다름 log.info("result={}", result2);
			//공통 로직2 종료 }

      @Slf4j
      static class Hello {
          public String callA() {
              log.info("callA");
              return "A";
          }
          public String callB() {
              log.info("callB");
					return "B"; }
} 
}
```

클래스의 메타정보를 얻는 방법)

```java
Class classHello = Class.forName("hello.proxy.jdkdynamic.ReflectionTest$Hello");
```

```java
@Test
void reflection1() throws Exception {
//클래스 정보
        Class classHello =  Class.forName("hello.proxy.jdkdynamic.ReflectionTest$Hello");

				Hello target = new Hello(); //callA 메서드 정보
        Method methodCallA = classHello.getMethod("callA");
        Object result1 = methodCallA.invoke(target);
        log.info("result1={}", result1);
//callB 메서드 정보
Method methodCallB = classHello.getMethod("callB"); Object result2 = methodCallB.invoke(target); log.info("result2={}", result2);
}
```

- 효과)문자로 바꿈으로써, 이후 파라미터로 넘겨서 동적으로 변경이 가능함
- 왜 굳이 이렇게 하는가?
    
    위처럼 동적으로 변경이 가능라도록하여 “공통로직”을 처리할 수 있게함.
    

코드 변경

```java
@Test
  void reflection2() throws Exception {
      Class classHello = Class.forName("hello.proxy.jdkdynamic.ReflectionTest$Hello");
      Hello target = new Hello();
      Method methodCallA = classHello.getMethod("callA");
      dynamicCall(methodCallA, target);
      Method methodCallB = classHello.getMethod("callB");
      dynamicCall(methodCallB, target);
  }
  private void dynamicCall(Method method, Object target) throws Exception {
      log.info("start");
      Object result = method.invoke(target);
      log.info("result={}", result);
```

- 다만 리플렉션을 사용하지 않도록 해야함.
    
    string으로 넘기면 잘못작성했을 때 컴파일 단계가 아니여서 컴파일 오류는 발생하지 않고 런타임에서 발생한다.
    
    가장 무서운 오류는 사용자가 직접 실행할 때 발생하는 런타임 오류.
    

### JDK 동적 프록시

> 개발자가 직접 프록시 클래스를 만들지 않아도되고 이름 그대로 프록시 객체를 동적으로 런타임 개발자 대신 만들어주는 프록시이다.
> 

- 다만, 인터페이스를 기반으로 작성해야한다. → 인터페이스 필수

- ex)

프록시에 적용할 로직 작성

`InvocationHandler`    을 활용 → JDK의 규칙이라고 생각하면 됨.

<틀>

```java
public interface InvocationHandler {
       

public Object invoke(Object proxy, Method method, Object[] args)
          throws Throwable;
}
```

<적용>

```java
@Slf4j
public class TimeInvocationHandler implements InvocationHandler {
      private final Object target;
      public TimeInvocationHandler(Object target) {
          this.target = target;
}

@Override
 public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
			log.info("TimeProxy 실행");
			long startTime = System.currentTimeMillis();

			Object result = method.invoke(target, args);

			long endTime = System.currentTimeMillis();
			long resultTime = endTime - startTime; 
log.info("TimeProxy 종료 resultTime={}", resultTime); 
return result;
	}
}
```

<Test>

- 동적인 프록시 생성법

AInterface가 존재하고 그 구현체가 AImpl일때

```java

AInterface target = new AImpl();
TimeInvocationHandler handler = new TimeInvocationHandler(target);
AInterface proxy = (AInterface)Proxy.newProxyInstance(AInterface.class.getClassLoader(), new Class[]{AInterface.class}, handler);
proxy.call();

```

이때, 우리는 직접 프록시 클래스를 만들었나? 아니다. 

개발자가 직접 프록시 클래스를 만들지 않아도된다는 장점이있다.


- 결론


- logTrace에 적용해보기)
1. `InvocationHandler` 을 위임받아서 `LogTraceBasicHandler` 를 정의

```java

@Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
      TraceStatus status = null;
      try {
          String message = method.getDeclaringClass().getSimpleName() + "."
                  + method.getName() + "()";
					status = logTrace.begin(message); 

					//로직 호출
          Object result = method.invoke(target, args);
          logTrace.end(status);

          return result;
      } catch (Exception e) {
          logTrace.exception(status, e);
					throw e; }
}
```

1. 동적프록시 수동빈등록

```java
public class DynamicProxyBasicConfig {
     
		 @Bean
     public OrderControllerV1 orderControllerV1(LogTrace logTrace) {
          OrderControllerV1 orderController = new
				  OrderControllerV1Impl(orderServiceV1(logTrace));
          OrderControllerV1 proxy = (OrderControllerV1)
				  Proxy.newProxyInstance(OrderControllerV1.class.getClassLoader(),
                  new Class[]{OrderControllerV1.class},
                  new LogTraceBasicHandler(orderController, logTrace)
					);
          return proxy;
				}

			@Bean
      public OrderServiceV1 orderServiceV1(LogTrace logTrace) {
          OrderServiceV1 orderService = new
				  OrderServiceV1Impl(orderRepositoryV1(logTrace));
          OrderServiceV1 proxy = (OrderServiceV1)
				  Proxy.newProxyInstance(OrderServiceV1.class.getClassLoader(),
					new Class[]{OrderServiceV1.class},new LogTraceBasicHandler(orderService, logTrace)
					);
					return proxy;
			}
			

			@Bean
      public OrderRepositoryV1 orderRepositoryV1(LogTrace logTrace) {
          OrderRepositoryV1 orderRepository = new OrderRepositoryV1Impl();
          OrderRepositoryV1 proxy = (OrderRepositoryV1)
				  Proxy.newProxyInstance(OrderRepositoryV1.class.getClassLoader(),
                  new Class[]{OrderRepositoryV1.class},
                  new LogTraceBasicHandler(orderRepository, logTrace)
						);
          return proxy;
      }

```

1. no-log호출해보기 →log가 찍히는 문제가 발생

→ 이러한 문제를 해결하기 위해서 **메서드 이름을 기준으로 특정 조건을 만족하면** 로그를 남기도록 한다.

<handler변경>

```java
//메서드 이름 필터
String methodName = method.getName();
if (!PatternMatchUtils.simpleMatch(patterns, methodName)) {
      return method.invoke(target, args);
}
```

메서드 이름 필터를 적용한 부분을 invoke안에서 실행시킴으로써 `LogTraceFilterHandler` 를 작성한다.

`LogTraceFilterHandler` ⇒ 이름이 매칭되지 않다면 실제 로직을 바로 호출해서 로그로직을(프록시처리부분)을 건너뛰도록 작성된 handler

<config 변경>

- pattern 만들기

```java
private static final String[] PATTERNS = {"request*", "order*", "save*"};
```

- filter를 적용한 handler를 실행하도록 proxy를 생성 → 아래는 config에서 service code만 변경한 예제

```java
@Bean
public OrderServiceV1 orderServiceV1(LogTrace logTrace) {
        OrderServiceV1 orderService = new
				OrderServiceV1Impl(orderRepositoryV1(logTrace));
        OrderServiceV1 proxy = (OrderServiceV1)
				Proxy.newProxyInstance(OrderServiceV1.class.getClassLoader(),
                new Class[]{OrderServiceV1.class},
                new **LogTraceFilterHandler**(orderService, logTrace, PATTERNS)
);
        return proxy;
    }
```

위와 같이 수정을 완료했다. 하지만 인터페이스가 있을 때만 사용이 가능한데, 클래스만 있을 때는 어떻게 해야하나?

### CGLIB

> 바이트코드를 조작해서 동적으로 클래스를 생성하는 기술을 제공한다.
> 

- 사싫 직접 사용하는 경우는 없어서 이해만 하면 됨

ex)

<틀>

`MethodInterceptor` 를 사용

```java
public interface MethodInterceptor extends **Callback** {
      Object intercept(Object obj, Method method, Object[] args, MethodProxy
  proxy) throws Throwable;
 }

/*
obj : CGLIB가 적용된 객체
method : 호출된 메서드
args : 메서드를 호출하면서 전달된 인수 
proxy : 메서드 호출에 사용
*/
```

- method를 사용해 invoke하는 것보다 MethodProxy 를 권장(성능이 더 좋음)

<적용>

```java
public Object intercept(Object obj, Method method, Object[] args,
	  MethodProxy proxy) throws Throwable {
		log.info("TimeProxy 실행");
		long startTime = System.currentTimeMillis();

	  **Object result = proxy.invoke(target, args);**

		long endTime = System.currentTimeMillis();
		long resultTime = endTime - startTime; log.info("TimeProxy 종료 resultTime={}", resultTime); return result;
}
```

<test>

`Enhancer` 를 사용해서 CGLIB는 프록시를 생성한다.

- 현재 상황 → `ConcreteService()` 가 존재하는데 인터페이스가 없을 때 →상속을 상황해야하는 상황
- 만약, class를 통해 프록시를 생성하고 싶다면 **상속을 이용해야함.**
    
    따라서, Target을 상속받은 class를 프록시의 SuperClass로 넣어줘야한다.
    

```java
@Test
  void cglib() {
      ConcreteService target = new ConcreteService();
      Enhancer enhancer = new Enhancer();
      enhancer.setSuperclass(ConcreteService.class);
      enhancer.setCallback(new TimeMethodInterceptor(target));
      ConcreteService proxy = (ConcreteService)enhancer.**create();**
      log.info("targetClass={}", target.getClass());
      log.info("proxyClass={}", proxy.getClass());
      proxy.call();
  }
```

<정리>


- 동적프록시는 구체클래스를 상속받아서 만들어진다.
- 이때, CGLIB 동적 프록시는 MethodInterceptor handler를 호출하여 부가기능을 처리한다.
- 제약사항 → 상속을 사용하기 때문에 제약사항이존재
    
    부모 클래스의 생성자를 체크해야한다.
    
    자식 클래스를 동적으로 생성해서 기본 생성자가 필요하다.
    

### 남은 문제

인터페이스가 없을 때는 JDK를 사용하고 클래스만 존재하면 CGLIB를 사용하도록 적용하려면?

두 기술을 함께 사용할 때 문제가 될 수 있다.

각각 InvocationHandler 와 MethodInterceptor 룰 중복으로 만들어서 관리해야할까?

특정 조건에 맞을 때 프록시 로직을 적용하는 기능도 공통으로 제공하는 두개를 합쳐준 것은 없나? → 프록시 팩토리의 역할.