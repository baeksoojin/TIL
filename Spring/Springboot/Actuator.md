# Actuator

## 프로덕션 준비 기능

> 운영 환경에서 서비스를 할 때 필요한 서비스에 문제가 없는지 모니터링하고 지표들을 감시하기 위한 기능을 프로덕션 준비 기능이라고한다.
> 

- 운영환경에서는 문제상황을 대비해서 프로덕션을 준비 기능을 제공해야한다.
    
    
    - metric(지표), trace(추적), auditing(감사 : login, logout 기록 등을 추적)
    - 모니터링 : 시스템의 상태를 지표를 통해서 확인

- connection pool을 얼마나 사용하고 있는지 등등, 애플리케이션이 잘 살아있는지 확인

스프링 부트는 액추에이터를 제공해서 프로덕션 준비 기능을 매우 편리하게 사용할 수 있도록한다.

---

## Actuator 사용

- setting

```java
implementation 'org.springframework.boot:spring-boot-starter-actuator' // actuator 추가
```

- actuator의 동작

```java
localhost/actuator
```

```java
{
	"_links": {
	      "self": {
	        "href": "http://localhost:8080/actuator",
	        "templated": false
	      },
	      "health-path": {
	        "href": "http://localhost:8080/actuator/health/{*path}",
	        "templated": true
	      },
	      "**health**": {
	        "href": "http://localhost:8080/actuator/health",
	        "templated": false
				}
	}
}
```

/health : 서버가 건강하게 살아있는지 체크

- yaml file로 actuator가 사용하는 기능들을 등록해서 사용가능

```java
management:
    endpoints:
      web:
        exposure:
					include: "*"
```

⇒ * 로 모든 기능을 노출시킨다는 의미.

### endpoint setting

- 활성화 → endpoint가 제공하는 기능을 사용할지말지를 선택  : 보통 다 활성화 되어있음(서버를 내리는 `shutdown` 제외)
- 노출 → 활성화한 엔드포인트를 HTTP로 노출할지 JMX로 노출할지를 선택(둘다 선택 가능)  : 위의 application.yml에서 처럼 노출시킬 것을 선택

- 만약에 `shutdown` 을 활성화하기 위해서는?

```java
management:
    endpoint:
      shutdown:
        enabled: true
    endpoints:
      web:
        exposure:
          include: "*"
```

서버가 살아있을 때 shutdown endpoint에 “POST”를 통해서 접근하면 살아있던 tomcat 서버가 다운된다.

- 노출하고 싶다면? `include`
- 노출하고 싶지 않다면? `exclude`

## 다양한 엔드포인트

springboot를 사용하면 다양한 엔드포인트를 제공해줘서 좋다. 그냥 사용하면 되니까.

- beans : 스프링 컨테이너에 등록된 빈을 보여준다.
- conditions : condition이 먹혔는지 안 먹혔는지 알 수 있음
- configprops : `@ConfigurationProperties` 에 대한 정보를 보여준다.
- env : 환경변수에 대한 정보들을 보여준다.
- **health** : 애플리케이션 헬스 정보를 보여준다.
- **httpexchanges** : HTTP 호출 응답 정보를 보여준다.
    
    HttpExchangeRepository 를 구현한 빈을 별도로등록해야 한다.
    
- **info** : 애플리케이션 정보를 보여준다.
- **loggers** : 애플리케이션 로거 설정을 보여주고 변경도 할 수 있다.
- **metrics** : 애플리케이션의 메트릭 정보를 보여준다.
→ cpu usage, memory usage 등등을 확인가능
- mappings : @RequestMapping 정보를 보여준다.
- threaddump : 쓰레드 덤프를 실행해서 보여준다.
- shutdown : 애플리케이션을 종료한다. 이 기능은 **기본으로 비활성화** 되어 있다.

**health**

- 요청에 응답할 수 있는지
- 데이터베이스가 응답하는지
- 디스크 사용량에 문제가 없는지

```java
management:
    endpoint:
      health:
        show-details: always
				# show-components: always -> status에 대해서만 확인가능
```

→ 상세히 볼 수 있게되어 DB, disk에 대한 정보를 알 수 있다.

```java
{
    "status": "UP",
    "components": {
      "db": {
        "status": "UP",
        "details": {
          "database": "H2",
          "validationQuery": "isValid()"
        }
      },
      "diskSpace": {
        "status": "UP",
        "details": {
          "total": 994662584320,
          "free": 303418753024,
          "threshold": 10485760,
          "path": ".../spring-boot/actuator/actuator/.",
          "exists": true
} },
      "ping": {
        "status": "UP"
} }
}
```

- db : 애플리케이션이 DB에게 살아있는지 확인하는 valication 체크 기능을 확인하는데 정상이면 status : “UP”
- diskSpace : disk의 free, total등의 정보를 제공하고 정상이면 역시 status:”UP”을 제공
- status : “UP”이 되기 위해서는 db, disk 모두 정상이여야한다. 하나라도 문제라면 “DOWN”이 된다.

추가로 db, mongo, redis, disksapce, ping과 같은 데이터 요청 및 관리 등에 필요한 기능에 대한 헬스값을 제공한다.

**info**

애플리케이션의 기본 정보를 노출한다.

java, os, env, build, git 정보를 제공하는데 env, java, os는 기본으로 비활성화 되어있다.

처음에는 아무것도 없기에 개별항목에서 info정보를 추가한다.

- 활성화

```java
management:
    **info: # java, os는 기본으로 활성화 되지 않아서 활성화 시켜줘야함
      java:
        enabled: true
      os:
        enabled: true
			env:
        enabled: true**
		endpoint:
      health:
        show-details: always
				# show-components: always -> status에 대해서만 확인가능
    endpoints:
      web:
        exposure:
          include: "*"

**info: 
	app:
      name: hello-actuator
      company: yh**
```

endpoint의 하위가 아니라 바로 management아래에 작성한다.

- env를 확인하고 싶다면 info 밑에 어떤 것을 체크할 것인지 app, appname, company를 등록해서 세팅해준다.

- build 정보를 확인하고 싶다면 → 빌드정보가 `META-INF/build-info.properties` 에 들어있어야함
    
    ```java
    springBoot {
          buildInfo()
    } 
    ```
    
    build.gradle에 설정추가
    
- git 정보

plugin setting

```java
id "com.gorylenko.gradle-git-properties" version "2.4.1" //git info
```

이후 빌드를 하면 `build/resources/main/git.properties` 안에서 정보를 확인가능.

또한 url에서도 커밋 id와 main branch를 확인가능

**logger**

```java
@Slf4j
@RestController
public class LogController {
      @GetMapping("/log")
      public String log() {
          log.trace("trace log");
          log.debug("debug log");
          log.info("info log");
          log.warn("warn log");
          log.error("error log");
          return "ok";
} }
```

참고)내려갈수록 심각한 로그

⇒ 로거 사용을 위해서 로깅레벨을 설정

```java
logging:
    level:
			hello.controller: debug
```

url

- /loggers : 에 접속하면 스프링의 기본 레벨이 INFO여서 “**INFO**” 레벨에서 데이터들이 나옴
- hello.controller(패키지) : 우리가 위에서 만든 controller는 “**DEBUG**”로 나옴
- /loggers/hello.controller
    
    : hello.controller의 로그만 따로 접근가능
    

**실시간 로그레벨 변경**

- 보고싶은 level로 변경할 수 있는데 url에 “POST” action

운영 서버는 보통 중요하다고 판단되는 “INFO”로그 레벨까지만 사용한다. 이때, **갑자기 문제가 발생해서 “DEBUG” or “TRACE”**까지 보고싶다면? 일반적으로는 설정을 변경하고 서버 다시시작이 필요하다.

⇒ **loggers endpoint를 사용하면 실시간으로 로그 레벨을 변경할 수 있다.**

endpoint의 action을 `“POST”`로 해서 body에 json으로 configuredLevel을 TRACE로 넘길 수 있다.

```java
{
      "configuredLevel": "TRACE"
}
```

**httpexchanges**

http 요청과 응답의 과거 기록 확인

- `HttpExchangeRepository` 인터페이스의 구현체를 빈으로 등록해야 사용가능

ex)

- `InMemoryHttpExchangeRepository` 를 제공해주는데, 이것을 빈으로 등록해서 사용하면 http요청이 과거부터 현재까지 얼만큼 쌓여있는지를 판단해준다.
- 최대 100개의 요청이 디폴트이고 넘어간다면 과거의 요청을 삭제해서 메모리를 관리해준다.

### 보안

- actuator를 통해서 알 수 있는 정보가 너무 많기 때문에 외부망에서 사용할 수 있도록 하면 안 된다.
    
    ⇒ 외부인터넷에서 접근이 불가능하도록 막아야한다.
    

**포트분리**

- server의 port를 변경해서 actuator를 다른 포트로 접근하게끔한다.

혹시 외부망에서 접근을 해야하는 경우라면, 인증을 사용한다.

**인증**

스프링 인터셉터 또는 스프링 security를 사용해서 인증된 사용자만 접근 가능하도록 추가 개발을 한다.