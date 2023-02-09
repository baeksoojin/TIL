# Spring을 활용한 HTTP header, 기본정보조회

## HTTP header, 기본정보조회

헤더는 httpmethod, locale, cookie 등의 정보를 담고 있다. <br>
이때, ReqeustHeader를 MultiValueMap을 통해서 읽을 수 있고 header에 담긴 모든 정보를 출력한다.<br>

- ex code<br>
```
@Slf4j
@RestController
public class RequestHeaderController {

    @RequestMapping("/headers")
    public String headers(HttpServletRequest request,
                          HttpServletResponse response,
                          HttpMethod httpMethod,
                          Locale locale,
                          @RequestHeader MultiValueMap<String, String> headerMap,
                          @RequestHeader("host") String host,
                          @CookieValue(value = "myCookie", required = false) String cookie
                          ){
                        log.info("request={}", request);
                        log.info("response={}", response);
                        log.info("httpMethod={}", httpMethod);
                        log.info("locale={}", locale);
                        log.info("headerMap={}", headerMap);
                        log.info("header host={}", host);
                        log.info("myCookie={}", cookie);
        return "ok";
    }

}
```

**return type을 string으로 하지 않고 다른 타입으로도 사용이 가능하다.**<br>
그때그때 필요하면 공식문서를 참고하면 될 것이다.<br>
> https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-methods

**살펴보면 inputstream(reader), outputstream(writer)도 파라미터로 들어갈 수 있음**을 알 수 있는데 이것은 이후 body 데이터를 처리하는 과정에서 많이 사용되기에 기억해둬야한다!<br>


