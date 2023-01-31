# URI

> URI란 Uniform Resource Identifier로 resource 식별을 위한 종합된 방법이다.

URI는 Locator, Name 또는 둘다 추가로 분류된다.<br>
URI안에 Locator를 통해서 식별하는 방법 URL과 Name을 통해서 식별하는 URN이 존재한다.<br>

보통 URN으로는 모든 것을 다 매핑할 수 없으니 잘 사용하지 않고 URL을 사용한다.

-----

## URL

1. scheme : 주로 protocol을 사용. 어떤 방식으로 자원에 접근할 것인가하는 약속 규칙(http, https, ftp)
2. userinfo : 사용자인증정보를 포함해서 인증해야할 때 사용하지만 거의 사용하지 않음
3. host name : domain name or ip address
4. port number : 일반적으로 생략하나 http는 80, https는 443를 사용
5. path : 리소스 경로로 계층적 구조로 /home/100, /members 등에 해당
6. query : ?로 시작하고 &로 추가적으로 key=value형태의 query가 가능한데 query parameter, query string으로 불림

