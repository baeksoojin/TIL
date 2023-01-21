# Springboot에서의 Servelt실행 이해하기

## Springboot Servlet 환경 구성

- @ServletComponentScan<br>
현재 나의 패키지 안에 등록된 Servlet을 모두 찾아서 자동으로 등록해준다.<br>

- WAS의 역할
WAS가 request, response 객체를 요청이 들어올때마다 새롭게 생성하며 servlet container에 등록된 servlet을 찾아서 던져주는 역할을 한다.<br>

```
@WebServlet(name ="helloServlet", urlPatterns = "/hello")
public class HelloServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException{
        System.out.println("HelloServlet.service");
        System.out.println("request = " + request);
        System.out.println("response = " + response);
    }

}
```

결과는
~~~
HelloServlet.service
request = org.apache.catalina.connector.RequestFacade@2c53dabd
response = org.apache.catalina.connector.ResponseFacade@43464f81
~~~
다음과 같이 나온다.<br>

-----
## request, response처리

- paramter 받기<br>
urlPatterns에 이어 query parameter를 이용해서 특정 변수와 값을 보내면 그 변수를 Servlet을 활용해서 편리하게 꺼낼 수 있다.<br>
```
request.getParameter("username");
```

- response 넘기기
header의 context-type 설정 및 body에 넘길 정보를 설정하는 등의 작업이 가능하고 client에 WAS를 통해서 전송된다.<br>
```
//header
response.setContentType("text/plain");
response.setCharacterEncoding("utf-8");
//body
response.getWriter().write("hello"+username);
```

아래의 image처럼 query parameter를 사용해서 username을 넘기면 설정해놓은 response를 client로 넘어가는 것을 확인 할 수 있다.<br>

<img width="426" alt="image" src="https://user-images.githubusercontent.com/74058047/213879632-bf5501ea-00b6-48e2-8a28-4391d8d8f18a.png">

-----
## 동작 방식 설명

**Browser <-> Springboot 내장톰켓서버<br>**

Springboot의 내장톰켓서버는 servlet container를 가진다.<br>
WAS가 requst, response의 새로운 객체를 만들면 특정 servlet container에게 전달해주고 servlet이 종료될때 reponse를 WAS에게 다시 넘긴다.<br>
이후 response는 Browser에게 전송된다.<br>

