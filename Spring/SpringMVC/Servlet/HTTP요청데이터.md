# HTTP 요청 데이터와 Servlet

## HttpServletRequest

HTTP 요청 메시지를 편리하게 조회할 수 있다.<br>
고객의 요청부터 응답까지 HTTP 요청 메세지를 관리해준다.<br>
- 저장 : request.setAttribute(name,value)<br>
- 조회 : request.getAttribute(name)<br>
session 관리 기능을 제공해주는데 `request.getSession(create: true)`를 통해서 가능하다.<br>

결국 servlet은 HTTP request, response에 대해서 편리하게 활용하기 위한 기술이기에, HTTP에 대해서 이해해야한다.<br>
먼저 요청 데이터의 형식을 알아보자!<br>

## HTTP 요청 데이터의 이해

1. GET - 쿼리 파라미터<br>
?부터 시작해서 url에 query parameter를 통해서 데이터를 전달하는 가정에서 주로 사용한다.

2. POST<br>
html form의 input tag에서 name항목으로 구분하여 데이터를 전달하는데 message body를 통해서 전달한다.<br>
Content-Type이 application/x-www-form-urlencoded 형식이됙 username=hello&age=20등을 예시로 들 수 있다.<br>
client의 form 작성이 필요한 회원가잉ㅂ, 상품 주문 등 data를 변경, 등록하는 과정에서 사용한다.<br>

3. HTTP message body<br>
HTTP API에서 주로 사용하는데 요즘에는 xml, text보다는 json을 주고 사용하고 post, put, patch를 모두 사용할 수 있다.

그렇다면 HTTP request, response를 servelt을 활용해서 어떻게 사용할 수 있을지에 대해서 알아보자!<br>

## HttpServeltRequest의 기본 사용법<br>

- annotation<br>
@WebServlet annotation을 활용한다.<br>
``` @WebServlet(name ="requestHeaderServlet", urlPatterns = "/request-header") ```
처럼 작성이 가능한데 /request-header를 들어가면 servelt container의 requestHeaderServlet이 작동한다.<br>

- `HttpServletRequest` interface 객체 request를 통한 HTTP request startline 및 헤더정보 조회방법<br>

    <몇가지 예시><br>
    1. header의 accept-language를 조회하고 싶다면?<br>
    `request.getLocales().asIterator()`를 통해서 정보에 접근하고 `forEachRemaining(locale -> ()))`를 통해서 모든 language에 접근이가능하다.<br>
    이때, 가장 우선순위를 가지는 언어를 조회하고 싶다면 `request.getLocale()`를 통해서 가능하다.<br>

    2. cookie를 조회하고 싶다면?<br>
    cookie의 데이터를 처리하기 쉽게 기능을 제공해주는데,  `request.getCookies()`를 통해서 Cookie class를 통해서 다룰 수 있다.<br>

    3. content를 조회하고 싶다면?<br>
    `request.getConetentType()`을 사용하여 content-type을 조회할 수 있으며 `request.getContentlLength()`와 `request.getCharacterEncoding()`을 사용해서 길이와 인코딩 정보를 알 수 있다.<br>

    이외에도 remote, local정보도 조회가 가능하다.<br>

- HTTP 요청데이터가 전송되는 메서드에 따라서 `HttpServletRequest`객체를 활용하는 방법이 달라진다.<br>
    1. GET method일때는 parameter를 조회한다.<br>
    2. 

## GET method

1. asIterator().forEachRemaining()을 사용하여 모든 parameter에 접근이 가능하다.<br>

    ```
    req.getParameterNames().asIterator().forEachRemaining(paramName -> System.out.println(paramName+"="+
                    req.getParameter(paramName)));
    ```

2. `request.getParameter(name:"입력")`으로 단일 parameter에 접근한다.<br>

    ```
    String username = req.getParameter("username");
            String age = req.getParameter("age");

    ```

3. name이(parameter key값)이 동일하게 여러번 url을 통해 parmater로 들어온 경우 모두 조회하고 싶다면?<br>

    원래대로라면 내부에서 먼저잡힌 값이 들어오게 된다.<br>

    ` public String[] getParameterValues(String name);`
    다음과 같이 name을 통해서 String 배열을 반환하는 getParameterValues를 사용하여 모두 조회할 수 있다.<br>

    따라서 *http://localhost:8080/request-param?username=sujin&age=20&username=baek*의 url을 입력하여 GET method를 통해서 parameter를 넘기게 된다면 결과는 아래와 같다.<br>

    ![image](https://user-images.githubusercontent.com/74058047/216779695-877fbeea-4fbc-4f37-aaa6-4c865185b84b.png)

* 정리)<br> request.getParameter()는 중복조회가 되지 않아서 복수의 값을 갖는다면 사용해서는 안 되고 request.getParameterValues를 통해서 조회해야한다.<br>

## HTML FORM과 POST method 데이터 처리

form data가 전송될 때의 header에 담긴 content-type을 보면 *application/x-www-form-urlencoded*인 것을 확인 할 수 있다.<br>
특징은 message body에 useranme=hello&age=20의 형태가 위의 GET method를 통해서 **url로 query parameter를 통해서 data를 전송했을 때와 동일하다느 것이다.**

* 정리)<br> 쿼리 파라미터 조회 메서드를 그대로 사용하여 POST method를 통해서 message body로 들어온 data를 조회하면 된다.<br>

`request.getParameter()`는 GET의 URL의 쿼리파라미터, POST의 HTML Form 형식 모두 지원한다.

## API message(json)

단순 text도 가능하지만 json으로 많이 사용한다.<br>
![jackson](https://user-images.githubusercontent.com/74058047/216781597-703e241b-87ae-462b-9cd1-da5d910d3690.png)

- jackson library 사용
*springboot가 기본으로 제공*하는 Jackson lib를 사용해서 json data를 가공할 수 있다.<br>
넘어온 json data를 `ObjectMapper()`를 사용해서 만들어놓은 class의 변수에 맞춰서 value를 저장하였다가 읽을 수 있다.

<예제><br>
    1. HelloData라는 class를 생성(username, age 존재)
    2. objectMapper.readValue(messageBody[json data], HelloDataa.class[tupe])를 통해서 값을 세팅
    3. getter를 사용해서 데이터를 사용

    ```
    @Override
        protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

            ServletInputStream inputStream = req.getInputStream();
            String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

            System.out.println("messageBody = "+messageBody);

            HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);

            System.out.println("helloData.username = "+ helloData.getUsername());
            System.out.println("helloData.age = "+helloData.getAge());


            resp.getWriter().write("ok");

        }
    ```

    ![objectMapper](https://user-images.githubusercontent.com/74058047/216781934-3446eaec-7487-4932-b7c9-538f9f359a65.png)

+ 참고)<br>
form data도 message body로 전송돼서 string을 직접 읽을 수 있지만 굳이 그렇게 하지 않는다.<br>
