# Logger

## slf4j

> SLF4J는 Simple Logging Facade for Java로 기본으로 제공하는 java.util.logging, logback, log4j와 같은 다양한 로깅 프레임워크에 대한 인터페이스를 제공하는 라이브러리이다.<br>

따라서 추상화 프레임워크이기에 단독으로 사용되지 않고 원하는 로그 프레임워크와 함께 사용하면 된다.<br>

## logger 생성 테스트

```
@RestController
public class LogTestController {
    private final Logger log = LoggerFactory.getLogger(getClass());

    @RequestMapping("log-test")
    public String logTest(){
        String name ="Spring";

        System.out.println("name = "+name);
        log.info("info log={}",name);
        return "ok";
    }

}
```

code 실행시,<br>

    name = Spring
    2023-02-09 06:58:04.708  INFO 70820 --- [nio-8080-exec-1] t.springmvc.basic.LogTestController      : info log=Spring

name = Spring처럼 print한것을 찍히게 하는 것보다 logger를 사용하는게 더 많은 정보를 디테일하게 얻을 수 있다.<br>
다음과 같이 Log가 찍히는 것을 알 수 있는데 `LoggerFactory.getLogger`를 통해서 Class정보를 찍게끔 등록해놨기에 `log.info`를 호출하면 해당 thread와 현재의 class에 대한 정보를 출력해준다.<br>

## 사실 로거를 사용하는 핵심은 "로그레벨"분리에 있다. 로그레벨을 분리함으로써 구조화된 로그정보

```
 //log level설정을 통한 구조화된 로깅을 진행
log.trace("trace log={}",name);
log.debug("debug log={}",name);//개발서버에서 보는 로그정보
log.info("info log={}",name);
log.warn("warn log={}",name);
log.error("error log={}",name);
```

- **어떠한 것들만 볼 것인가?** 에 대해서 설정하고 싶다면 `logging.level.tutorial.springmvc`의 하위의 패키지에서 사용할 로거 정보를 설정해야한다.<br>
resources>application.properties에 들어가서 확인할 레벨의 **시작시점**을 설정한다.<br>

- ex)<br>
`logging.level.tutorial.springmvc=trace` 라면 trace부터 시작해서 error level까지 보는 것이고, `logging.level.tutorial.springmvc=info`라면 info~error까지 본다는 것을 의미한다.<br>

- 사용상황)<br>
traffic이 많은 운영서버는 `info`를 활용<br>
디버깅이 필요한 개발 서버는 `debug`를 활용<br>

## lombok 활용

```

@Slf4j
@RestController
public class LogTestController {
//    private final Logger log = LoggerFactory.getLogger(getClass());

//이하내용생략(위와 동일)
}
```

`@Slf4j`를 활용하면 `private final Logger log = LoggerFactory.getLogger(getClass());`을 사용하지 않아도 이미 설정되어있기에 log를 바로 사용하면 된다.<br>


## 올바른 사용법

`log.trace("trace my log="+name);`처럼 사용하면 안 된다.<br>
**java는 `+`를 보고 문자열을 합하는 연산** 을 무조건 실행하게 되는데 log level을 trace로 시작하지 않는다면 log를 출력하지도 않으면서 **쓸모없는 연산과 메모리만 증가**시키는 것이다.<br>

따라서, parameter를 넘기는 방법을 사용한다.<br>
