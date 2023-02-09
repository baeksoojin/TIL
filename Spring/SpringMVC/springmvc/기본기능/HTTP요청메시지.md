# HTTP 요청 메시지

## 텍스트일때

body에 저장된 데이터를 읽어오고 싶다면 inputStream을 사용한다.<br>

```
@PostMapping("/request-body-string-v1")
public void requestBodyString(HttpServletRequest request,
                                HttpServletResponse response) throws IOException {
    ServletInputStream inputStream = request.getInputStream();
    String messageBody = StreamUtils.copyToString(inputStream,StandardCharsets.UTF_8);
    log.info("messageBody={}", messageBody);
    response.getWriter().write("ok");
}
```

다만, 여기서는 굳이 HttpServletRequest까지 사용하지 않아도 되는데 사용한다는 점에서 개선사항이 보이고 HttpServletResponse도 사용하지 않아도 되는데 사용한다는 점이 보인다.<br>

<개선코드>


힌트) <br>
request,response를 굳이 사용하지 않아도 될때는 필요한 데이터만 hasmap을 사용해서 넘기는 것처럼 원하는 것만 파라미터로 넘길 수 있게 제공한다.<br>
그렇다면, 공식문서에 들어가서 단순 입,출력을 위한 파라미터제공을 해주는지 체크한다.<br>

SpringMVC는 InputStream, OutputStream(각각 reader, writer)를 파라미터로 제공한다.<br>


```
@PostMapping("/request-body-string-v2")
public void requestBodyStringV2(InputStream inputStream, Writer responseWriter) throws IOException {
    String messageBody = StreamUtils.copyToString(inputStream,StandardCharsets.UTF_8);
    log.info("messageBody={}", messageBody);
    responseWriter.write("ok");
}
```

### 하지만, spring은 message converter를 제공해서 위의 과정을 진행하지 않아도 된다. 


<더 개선된 코드>
```
@PostMapping("/request-body-string-v3")
public HttpEntity<String> requestBodyStringV3(HttpEntity<String> httpEntity) {
    String messageBody = httpEntity.getBody();
    log.info("messageBody={}", messageBody);
    return new HttpEntity<>("ok");
}
```
springMVC는 HttpEntity parameter를 지원한다.<br>

- httpEntity<br>
메시지 바디 정보를 직접 조회

- 응답시에도 사용<br>
메시지 바디 정보 직접 반환<br>
view 조회를 하지 않고 응답 메시지를 바로 넣어버리는 로직으로 처리된다.<br>

- ResponseEntity

만약에 **상태코드를 넣고 싶다면** `RequestEntity`를 사용하여 `return new ResponseEntity<String>("message",HttpStatus.CREATED);`등과 같이 사용할 수 있다.<br>

### 하지만, spring은 @RequestBody annotation을 제공해서 손쉽게 데이터를 읽는다.<br>

@RequestBody 를 사용하면 HTTP 메시지 바디 정보를 편리하게 조회할 수 있다.<br>

요청 올 때 : `@RequestBody`(조회시 사용)
응답 할 때 : `@ResponseBody`(응답시 사용하고 뷰 리졸버를 거치지 않음.)

```
@ResponseBody
@PostMapping("/request-body-string-v4")
public String requestBodyStringV4(@RequestBody String messageBody) {
    log.info("messageBody={}", messageBody);
    return "ok";
}
```

## Json일때

1. json을 객체로 돌리기<br>
`private ObjectMapper objectMapper = new ObjectMapper();`로 객체로 전환한다.<br>
json을 읽은 다음에 사용한다.<br>
다만, `HTTP Message Converter`를 사용하면 위의 과정을 자동으로 해준다.<br>

2. json 읽어오기<br>

```
@ResponseBody
@PostMapping("/request-body-json-v3")
public String requestBodyJsonV3(@RequestBody HelloData data) {
    log.info("username={}, age={}", data.getUsername(), data.getAge());
    return "ok";
}
```

- `requestBody`에 직접 만든 객체를 지정 할 수 있다.<br>
- `Http Message Converter`가 Json을 객체로 변환하는 과정을 대신 처리해준다.

- `@RequesstBody`는 생략불가능

spring에서는 생략하면 `@ModelAttribute`가 동작함. 따라서 생략하면 요청 파라미터를 처리하게 돼서 생략 불가능하다.<br>