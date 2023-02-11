# 응답에서의 HTTP Message Converter : @ReponseBody annotation의 활용

HTTP Message Converter를 통해서 요청 메시지를 처리하고 응답을 하는 과정을 편하게 진행할 수 있다.<br>
기존에는 inputstream, StreamUtils를 사용해서 전체를 byte 단위로 읽은 후, 인코딩을 진행하는 과정을 통해서 요청 데이터를 읽고 처리한 후에 반환했다면 그 과정을 **Converter**의 역할이 내제된 `@ResponseBody` annotation을 사용할 수 있다.<br>

--------

## Message Converter
말그대로 message를 전환해주는 것으로 필요한 형태로 변환하기 위해서 사용한다.<br>
`String messageBody = StreamUtils.copyToString(inputStream,StandardCharsets.UTF_8);`의 경우처럼 byte code를 string으로 변환해서 messageBody에 담긴 json data를 읽어오는 과정을 처리하는 경우 등에서 사용된다.<br>

---------

## [what] @ResponseBody

> ResponseBody는 view template처럼 html을 생성해서 응답하는 것이 아닌, HTTP API처럼 json 데이터를 HTTP message body에서 직접 읽거나 쓰는 경우, HTTP message converter를 사용할 때 활용하는 annotation이다.


### 특징

- HTTP Body에 문자 내용을 직접 반환하여 `viewResolver`를 사용하지 않는다.<br>
- `HttpMessageConverter`를 동작시킨다.<br>
    1. 기본 문자처리 : `StringHttpMessageConverter`
    2. 기본 객체처리 : `MappingJackson2HttpConveter`
    byte 처리 등의 여러 HttpMessageConverter가 **기본으로 등록되어 있다.**<br>

    response data는 HTTP Accept 헤더와 서버의 컨트롤러 반환 타입 정보를 조합하여 `HttpMessageConverter`가 선택된다.<br>

- 스프링 부트에서 등록된 기본 메시지 컨버터
    0. ByteArrayHttpMessageConverter
    1. StringHttpMessageConverter
    2. NappingJackson2HttpConverter

-----

## [how] Http Message Converter의 동작원리 및 사용원리

### 동작원리

<img width="716" alt="image" src="https://user-images.githubusercontent.com/74058047/218240885-061d4ae9-d328-49e3-bc10-d92e42c3e0c5.png">

1. 브라우저의 요청(reqeust)<br>
2. controller의 호출<br>
3. @ResponseBody의 발견 동시에, HttpMessageConver의 동작<br>

### 사용원리

@Responsebody로 값이 반환되고, response여부를 체크하고 **조건을 만족하면 write()를 호출해서 HTTP 응답 메시지 바디에 데이터를 생성한다.**<br>
1. 대상 클래스 타입을 지원하는지 체크하며 사용할 메시지 컨버터를 선택<br>
2. Client가 받아들일 수 있는가? Accept type을 지원하는지 체크<br>

- `canWrite()`의 호출<br>
메시지 컨버터가 메시지를 쓸 수 있는지 여부 체크를 위해서 호출되는데 <br>
    - 대상 클래스 타입을 지원하는지 체크(return의 대상 클래스의 type이 `byte[]`인지 `String`인지 `Data객체`인지 체크한다.)<br>
        이때, 각 타입별로 지원하는 메시지 컨버터가 선택되고 아래의 체크를 하기 위해서 넘어간다.<br>
    - Accept media type을 지원하는지 체크<br>
        `text/plain`, `application.json`, `*/*`(모두가능) 등을 체크하고 쓸 수 있는지 체크한다.<br>

- 예를 들어서 체크해보자 <br>
    ```
    @ResponseStatus //ResponseEntity처럼 응답코드 사용을 위해 적용
    //@ResponseBody //mesage converter사용 -> class의 annotation으로 @RestController를 사용중
    @GetMapping("/response-body-json2")
    public HelloData responseBodyJsonV2(){
        HelloData helloData = new HelloData();
        helloData.setUsername("userB");
        helloData.setAge(20);
        return helloData;
    }
    ```
    
    1. return type이 HelloData로 class type<br>
         => application/json data를 다루는 `MappingJackson2HttpMessageConverter`가 선택된다.<br>
    2. Accept type이 application/json인지 체크<br>
        - text/html
        ![image](https://user-images.githubusercontent.com/74058047/218242686-a000b5f8-6031-4525-ab69-2332bc6c1328.png)<br>
        다음과 같이 `406 error`가 난 것을 확인할 수 있다.<br>
        -  참고로 Accept type은 `application/json`이 되어야한다.<br>

