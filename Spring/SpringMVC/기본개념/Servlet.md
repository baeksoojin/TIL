# Servlet

## 서버에서 처리해야 하는 업무

웹 브라우저에서 생성한 HTTP message를 처리하기 위해서 TCP/IP listen과정부터 시작해서 socket연결을 통한 message parsing의 과정 등등 수많은 과정을 거치고 TCP/IP에 response를 전달한 다음에 socket통신을 종료한다.<br>

1. TCP/IP 대기 및 소켓 연결
2. HTTP 요청 메시지를 parsing and read
3. POST 방식, save URL 인지
4. Content Type 확인
5. HTTP body parsing 및 데이터처리
6. 저장 프로세스
7. 비즈니스 로직 실행
    데이터베이스에 저장 요청(의미있는 비즈니스 로직)
8. HTTP 응답 메시지 생성 시작
9. TCP/IP 응답 전달 및 소켓 종료

## 서버의 업무를 대신 처리해주는 Servlet

**Server의 역할을 Servlet이 대신해준다.**<br>

- HTTP 요청 정보(name, email 등)를 편리하게 사용이 가능하다. HttpServeltRequest
- HTTP 응답 정보를 편리하게 제공할 수 있다. HttpServletResponse
- HTTP 스펙을 매우 편리하게 사용할 수 있다.

servlet container에서 servlet 객체를 생성하여 호출하고 생명주기를 관리해준다.<br>
이때, servlet을 지원하는 WAS를 서블릿 컨테이너라고한다. 대표적인 예로 Tomcat이 있다.<br>

- servlet 객체를 싱글톤으로 관리<br>
request마다 data가 달라서 request는 새롭게 만들지만 Servlet 객체를 새롭게 생성할 필요는 없기에 싱글톤으로 관리한다.<br>
최초 로딩 시점에 서블릿 객체를 미리 만들고 재활용을 한다.<br>
따라서 공유 변수 사용에 있어서 주의해야한다.<br>
- JSP도 서블릿으로 변환되어서 사용한다.
- Muli Thread 처리 지원이된다.
