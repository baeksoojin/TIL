# WEB SCOPE

## 웹스코프

<특징>

- 웹 환경에서만 동작하는 스코프를 의미한다.
- 프로토타입과 다르게 스프링이 해당 스코프의 종료시점까지 관리하고 종료 메서드가 호출된다.  

<종류>

1. request : HTTP 요청이 하나가 들어오고 나갈 때까지 유지되는 스코프로 각각의 HTTP 요청마다 별도의 빈 인스턴스가 생성되고 관리된다.
> request 요청이 들어온 다음 response를 반환할때까지 spring DI container에서 관리된다.

2. session : HTTP session과 동일한 생명주기를 가진다.
3. application : ServletContext와 동일한 생명주기를 가진다.
4. websocket : websocket과 동일한 생명주기를 가진다.


## Request scope

HTTP request에 맞춰서 각각 할당되는 request scope를 살펴보자!

각각 할당되니까 요청별로 property처럼 새로운 springbean이 생성되는 것인가?<br>
**그렇지 않다**<br>
각각 할당된다는 것의 의미는 Controller(springbean)에 의해서 client별로 전용 scope가 관리된다는 것을 의미하고 scope를 request가 무엇인지에 따라서 다르게 동작하는 것을 의미한다. 이후 client에게 response가 나갈 때 scope가 끝이 나게 된다.<br>

-------

## Request scope를 활용한 Logger

scope마다 처리되는 UUID가 다르기에 LOG 확인에 도움이 된다.
MyLogger Bean은 request 요청당 하나만 생성되기 때문에 초기화 메서드를 사용시에 uuid를 생성해서 넣어놓으면 다른 HTTP요청과 구분할 수 있다.
<br>

- intercepter에 구현하여 logger를 사용하면 좋을 것이다!!!<br>
- 다만, http요청 전에 service를 위해서 DI를 진행한다면 당연히 error가 날 것이다.
    아직 request가 없어서 scope가 할당되지 않았는데 DI를 진행하기 때문이다.
    따라서 provider를 함께 사용해줘야할 것이다.<br>
    provider조차 사용하고 싶지않다면? 프록시를 사용해도 된다.<br>

```
@Component
@Scope(value="request")
public class MyLogger {
}
```
다음과 같이 MyLogger를 생성하고 이를 웹의 controller, service에서 사용할 수 있다.<br>

1. provider를 사용한 Controller, Service에서 request DL
~~~
private final ObjectProvider<MyLogger> myLoggerObjectProvider;

@RequestMapping("log-demo")
@ResponseBody
public String logDemo(HttpServletRequest request){

    String requestURL = request.getRequestURL().toString();
    MyLogger myLogger = myLoggerObjectProvider.getObject();
    myLogger.setRequestURL(requestURL);

    myLogger.log("controller test");
    logDemoService.logic("testId");
    return "OK";

}
~~~
다음과 같이 Service, Container에 provider를 사용할 수 있다.<br>

2. Proxy를 사용한 방법

```
@Component
@Scope(value="request", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class MyLogger {
}
```

~~~
private final MyLogger myLogger;

@RequestMapping("log-demo")
@ResponseBody
public String logDemo(HttpServletRequest request){

    String requestURL = request.getRequestURL().toString();
    myLogger.setRequestURL(requestURL);

    myLogger.log("controller test");
    logDemoService.logic("testId");
    return "OK";

}
~~~

코드설명)<br>
가짜 proxycode를 만들어서 주입시켜준다. request와 상관없이 가짜 proxy class를 만들어서 주입시켜준다.<br>

    mylogger = class tutorial.core.common.MyLogger$$EnhancerBySpringCGLIB$$1a21a4b3<br>
    위와같이 spring이 조작하여 가짜를 넣어주고 실제로 request가 들어와 동작시작시 진짜를 넣어준다.<br>

    마치 singleton 보장을 위해서 configuration이 CGLIB를 통해서 bean으로 등록되는 것처럼 이때도 역시 CGLIB를 사용한다.<br>

- CGLIB을 사용하여 내 클래스 상속 받은 가짜 프록시 객체를 만들어 springBean에 등록한다.<br>
    가짜 프록시 객체는 요청이 들어오면 내부에서 진짜 빈을 요청하는 위임 로직이 존재하기 때문에 가능하다.<br>
    *가짜 프록시 객체는 원본 클래스를 상속 받아서 만들어졌기에 이 객체를 사용하는 client는 가짜 사실 여부와 관계 없이, 동일하게 사용이 가능하기에 **다형성**을 지키며 객체지향적으로 사용이 가능하다.<br>*
