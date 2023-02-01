# HTTP METHOD

## Method Type

1. GET : 리소스 조회
2. POST : 요청 데이터 처리, 주로 등록에 사용
3. PUT : 리소스 대체, 해당 리소스가 없다면 생성
4. PATCH : 리소스를 부분적으로 변경(회원 이름과 같이 특정 필드를 변경할 때 사용)
5. DELETE : 리소스 삭제

## GET

resource 조회 method로 url에 전재하는 자원을 요청할때 사용한다.<br>
서버에 요청하고 싶은 데이터를 query param을 통해서 전달할 수 있다.<br>
+)<br>
message body를 사용해서 데이터를 전달 할 수는 있지만, 지원하지 않는 서버들이 대부분이라 권장하지 않는다.<br>

1. 요청 HTTP Message전달
2. 서버가 message를 조회
3. 서버가 줄 데이터를 생성
4. 서버에서 응답 메시지를 생성해서 response를 client로 응답 HTTP Message전달

+) 조회할 때 서버끼리 "캐싱"하도록 처리되어있기에 조회할때 GET을 써야 유리하다.<br>

## POST

message body를 통해서 서버로 요청 데이터를 전달해 데이터를 처리하기 위한 요청을 할 때 사용한다.<br>
서버는 요청 데이터를 처리하게 된다.<br>

1. 요청 HTTP message를 body에 데이터를 포함하여 전달
2. 서버가 message를 조회
3. resource 처리
4. 서버에서 응답 메시지를 생성해서 response를 client로 응답 HTTP Message전달

+) 신규로 데이터를 생성하는 경우 응답데이터의 start-line에서 상태코드는 201번.<br>
+) 단순한 값의 변경을 넘어 프로세스의 상태가 변경되는 경우도 POST를 사용한다.<br>
+) control URI(resource로만 처리하기 어려운 경우)의 method로 주로 POST를 사용한다.<br>
    예를 들어, /orders/ 처럼 resource만 uri로 하고 method로 동작을 구분하는 것이 어려울 때가 있다. delivery를 시작하기처럼 동사를 method로만 처리하기 어려울 때 /orders/{orderId}/start-delivery 처럼 control uri를 만들게 되는데 이때 POST를 사용해서 상태를 변화시킨다.<br>

## PUT

resource를 덮어쓰는(대체) method로 없다면 생성하고 있다면 덮어쓴다.<br>

**특정 리소스의 위치를 알고있는 상태에서 resource를 다룰 때 사용한다.**
따라서 id값과 같은 pk값이 put method와 함께 넘겨진다.<br>

1. HTTP 요청 method 생성
2. server를 조회하고 "완전히"대체 혹은 생성
3. response data를 client에게 전달

+) 완전히 대체<br>

- step1)
username, age field가 존재
- step2)
put method로 age field만 전송
- step3)
age field만 존재

## PATCH

PUT처럼 덮어쓰는(대체) method로 없다면 생성하고 있다면 덮어쓴다.<br>
다만 다른 점은 **부분 변경**이 일어나 완전히 대체되는 것이 아니라는 점이다.<br>

+) 다만 PATCH를 지원하지 않는 경우가 있기에 그럴때는 POST를 사용하도록 하자.!!<br>

## DELETE

삭제를 위한 method이다.<br>



----------

## 안전성

>호출해도 리소스를 변경하지 않을때 안전하다고 하다.<br>

1. GET : 조회만 하기에 안전
2. POST, PUT, PATCH, DELETE : 리소스 변경이 일어나 안전하지 않음

+) 장애발생을 고려하는 것이 아니라 "리소스 변경"이 되냐/안되냐의 척도를 나타냄.<br>

## 멱등성

> 여러번 호출해도 결과가 같아야 멱등하다고 한다.<br>

1. GET, PUT, DELETE : 여러번 호출해도 같은 결과를 내보낸다.<br>
2. POST : 두번 호출하면 결과가 달라진다(결제를 두번호출하면 두번 결제됨).<br>

+) 멱등하다면(같은요청을 여러번해도 괜찮다면) 문제가 발생했을 때 자동으로 메서드를 다시 실행하여 자동복구 매커니즘에 적용이 가능하다.<br>
+) 재요청을 할 때 그 사이에 다른 method의 영향으로 데이터가 바껴있다면 리소스가 변경되어 다른 값이 나오는데(멱등하지 않게 나오는데) 멱등은 이러한 경우까지는 고려하지 않다. => 외부요인까지는 멱등성에서 고려하지 않는다.<br>

## Cacheable

> 이미 처리된 resource는 웹 브라우저가 저장하고 있을 때 캐시가능하다고 하다.

- GET, HEAD 정도만 캐시로 사용
url만 key로 잡고 cache를 하면 돼서 simple해서 실무에서 주로 사용한다.<br>


