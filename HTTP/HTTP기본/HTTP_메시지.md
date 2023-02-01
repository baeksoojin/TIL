# HTTP Message

## 메시지 구조

1. start-line
2. header
3. empty line(CRLF) 공백라인 필수
4. message body

## HTTP 메시지

1. 요청 메시지

    start-line : GET /search?q=hello&hl=ko HTTP/1.1<br>
    header : www.google.com<br>
    empty line<br>
    요청 메시지 body도 가질 수 있음<br>

- start-line<br>
request-line이 들어옴<br>
요청 메시지(GET/POST/PUT/DELETE) + 요청 대상(절대경로("/")[?쿼리]의 형태) + HTTP version
<br>

- header<br>
field-name(HOST): OWS(띄어쓰기 가능) field-value<br>




2. 응답 메시지

    start-line : HTTP/1.1 200 OK<br>
    header : Content-Type , Content-Length<br>
    empty line<br>
    message body(html 내용)<br>

- start-line<br>
status-line 들어옴<br>
HTTP version + HTTP 상태코드(200 성공 /400 클라이언트 요청오류/500 서버 오류) + 이유문구(사람이 이해할 수 있는 짧은 상태 글)

<br>

- header<br>
HTTP 전송에 필요한 모든 부가 정보를 포함<br>
Content-Type, Content-Length 등등

- Message Body<br>
실제 전송할 데이터를 포함(ex. html code)

