# HTTP API 설계

## api 설계 - post 기반 등록

회원 괸리 시스템을 만든다고 할 때 method 활용방법은 다음과 같다.

- 회원 목록 : /members (GET)
- 회원 등록 : /members (POST)
- 회원 조회 : /members/{id} (GET)
- 회원 수정 : /members/{id} (PATCH, PUT, POST 중 선택가능)<br>
기존의 리소스를 아예 덮어버리는 put <-> 특정 부분만 수정하고 싶은 patch<br>
**따라서 주로 patch를 사용하는 것이 바람직하다.**
- 회원 삭제 : /members/{id} (DELETE)

<특징><br>
클라이언트는 등록된 리소스의 새로운 신규 리소스 식별자가 포함된 uri를 모른다.<br>
따라서,Location에 response로 server에서 생성된 uri를 넘겨준다.<br>

### Collection<br>
- 서버가 관리하는 리소스 디렉토리
- 서버가 리소스의 uri를 생성하고 관리
- 회원 관리 시스템의 예시에서 collection은 /members에 해당한다<br>

-----


## api 설계 - put 기반 등록

파일 관리 시스템을 만든다고 할 때 method 활용방법은 다음과 같다.

- 파일 목록 : /files (GET)
- 파일 조회 : /files/{filenmae} (GET)
- 파일 등록 : /files/{filename} (PUT) : 파일이 없을 때는 등록하고 있다면 변경하는 클라이언트가 파일을 알고 있는 상태에서 진행하는 과정<br>
- 파일 삭제 : /files/{filename} (DELETE)
- 다른 기능(ex. 파일 대량 등록) : /files (POST)

<특징><br>
클라이언트가 리소스의 uri를 알고있어야한다.<br>
파일 등록 /files/{filename}과 같이 클라이언트가 filename을 이미 알고있어서 uri를 이미 알고 있고 그것을 변경하고 싶을 때 사용하는 것이다.<br>
ex) PUT /files/star.jpg<br>
<-> resource uri를 client가 모르고 있었던 POST와 차이가 있다.<br>

### Store
- client가 관리하는 resource 저장소라고한다.<br>
- 파일 관리 시스템에서 store는 /files에 해당한다.<br>

------

## 사용비중

> 실무에서는 대부분 POST를 사용하고 PUT을 사용하는 일은 거의 없다.<br>


------

## HTML FORM

- GET, POST만 지원이 가능하다.<br>
- html form을 사용해서 설계할때 control uri를 사용하게 된다.<br>

동사로 된 리소스 경로를 사용하게 되는데 이것을 control uri라고 한다.<br>
다만 무식하게 남발하면 안 되고 최대한 resource만으로만 안 될 때만 사용해야한다.<br>


