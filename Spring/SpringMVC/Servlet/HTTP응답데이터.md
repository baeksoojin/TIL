# HTTP 응답데이터와 Servlet

## HttpServletResponse HEADER 생성<br>
HttpServletResponse interface를 활용 instance인 resp를 활용해서 HEADER를 생성할 수 있다.<br>


1. Status생성<br>
HttpServletResponse.SC_[]를 통해서 status code를 넣어줄 수 있다.<br>
`resp.setStatus(HttpServletResponse.SC_OK);`처럼 사용이 가능하다.

2. response-header생성<br>
`resp.setHeader(name,value)`를 통해서 name에 해당하는 header를 value값으로 등록할 수 있다.<br>
기존의 header값을 설정해줄 수 있으며 내가 원하는 값도 전달이 가능하다.<br>

```
resp.setHeader("Content-Type","text/plain;charset=utf-8");
resp.setHeader("Cache-Control","no-cache, no-store, must-revalidate");
resp.setHeader("Pragma","no-cache");
resp.setHeader("my-header", "hello");
```

3. 결과<br>
1,2처럼 진행한 결과 localhost:8080/response-header에서 network 기록을 살펴보면 아래와 같은것을 확인 할 수 있다.<br>

<img width="840" alt="image" src="https://user-images.githubusercontent.com/74058047/216908327-43f24f45-230a-4bb0-8a23-725fa6a068f5.png">

<편의메서드><br>
1. content

```
 private void content(HttpServletResponse response) {
        
    response.setContentType("text/plain");
    response.setCharacterEncoding("utf-8");
    }
```
위처럼 response 객체의 편의 메서드를 활용하면 content를 name을 지정하지 않고 value만으로도 설정할 수 있다.<br>

2. cookie

```
private void cookie(HttpServletResponse response) {
        Cookie cookie = new Cookie("myCookie", "good");
        cookie.setMaxAge(600); //600초
        response.addCookie(cookie);
}
```

setHeader를 사용하지 않고 Cookie 객체를 생성해서 설정이 가능하다.<br>

3. redirect<br>
status를 302로하고 location을 form으로 보내고 싶다면 reponse 객체를 활용해서 setStatus, setHeader를 사용할 수 있지만 `sendRedirect(location)`을 통해서 redirection이 가능하다.<br>

```
private void redirect(HttpServletResponse response) throws IOException {
       
    response.sendRedirect("/basic/hello-form.html");
}
```

## 응답데이터

`PrintWriter` class를 활용해서 응답 데이터를 화면에 띄울 수 있는데 이것을 활용해서 데이터 응답 방법을 알아보자!<br>
그 방법은 총 3가지로 나눌 수 있다.<br>

1. text응답<br>

2. HTML응답<br>

```
@Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        resp.setContentType("text/html");
        resp.setCharacterEncoding("utf-8");

        PrintWriter writer = resp.getWriter();
        writer.println("<b>안녕</b>" );

}
```
HttpServletReponse 객체를 활용해서 ContentType과 CharacterEncoding방식을 설정할 수 있고 따라서 writer.println을 통해서 넘겨진 안녕은 굵은 글씨로 화면에 표시된다.<br>
<img width="1492" alt="image" src="https://user-images.githubusercontent.com/74058047/216915114-809ee639-3f06-4e7e-91a8-89d36e0f0d1a.png">

<br>

3. HTTP API - messagebody를 통한 json응답<br>

- `ObjectMapper`
objectMapper 객체를 사용해서 변환할 instance를 string으로 쉽게 변환이 가능하다.

- application/json;charset=uft-8<br>

- code<br>
```
@Override
protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    resp.setContentType("application/json");
    resp.setCharacterEncoding("utf-8");//참고로 json은 스펙상 utf-8로 설정되어있어서 하지 않아도 된다.

    HelloData helloData = new HelloData();
    helloData.setUsername("baek");
    helloData.setAge(20);

    String result = objectMapper.writeValueAsString(helloData);
    resp.getWriter().write(result);
}
```
<img width="1201" alt="image" src="https://user-images.githubusercontent.com/74058047/216916412-54ea882a-c18d-4382-90df-efcb0473b9a9.png"><br>
결과는 다음과 같이 Json data가 잘 나오는 화면에 찍어봄으로써 확인할 수 있다.<br>

