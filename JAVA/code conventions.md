# Java Code Convention
------

## 배경

[Code Conventions for the Java Programming Language](https://www.oracle.com/java/technologies/javase/codeconventions-contents.html) 를 참고 하였습니다.


clean-code를 위해, 가독성을 추구하여 직관적으로 이해할 가능성이 높은 코드로 만들어야한다.
<br>
- 가독성

코드는 read하는 것이 아니라 decode 해야한다.<br>
가독성은, 코드 해석에 드는 비용을 줄이는 작업을 확보하는 것을 의미한다.<br>

코드를 처음보는 사람은 팀원, 유지보수 후임, 오픈소스 혹은 API 사용자, 혹은 본인이 될 수 있다.<br>
그들의 코드 해석에 드는 비용을 줄여야한다.<br>

1. Legibility(표현적 가독성)

눈에 잘 들어오는 읽기 편한 코드

2. Readability(기능적 가독성)

변수, 함수, 클래스가 어떤 역할을 가지고 있으며 어떻게 동작하는지에 대해 직관적으로 알아보기 쉽게 작성한 코드

----

## java의 coding rule

[java 개발 가이드](http://developer.gaeasoft.co.kr/development-guide/java-guide/java-coding-style-guide/)를 참고 하고 있습니다.<br>

위의 룰을 지키기 위해서는 언어마다 정해진 디자인 방식이 존재하는데 이를 따라야한다.
<br>


### format

- 파일 인코딩시, UTF-8
- 들여쓰기는 tap대신 space 4개를 사용
- 중괄호 : 새로운 라인에서 시작하지 않고 제어문과 같은 라인, 제어문이 한줄이더라도 중괄호는 생략하지 않는다.

👍 good
```
if (isNull == false){
    System.out.println("hello!");
}
```

👎 bad
```
if (isNull == false)
{
    System.out.println("hello!");
}
```

```
if(isNull == flase) System.out.println("hello!");

```

- 띄어쓰기 : 메소드 이름 다음에는 띄어쓰기 없이 왼쪽 괄호를 사용한다. 배열다음 띄어쓰기 없이 괄호를 사용한다. 이전 연산자 간 양쪽에 띄어쓰기를 사용한다. 쉼표와 세미콜론 뒤에는 띄어쓰기를 사용한다. cast 사용시 공백을 사용한다.

👍 good
```
foo(i, j);
args[0];
z = 2 * x + 3 * y;
myMethod((byte) aNum, (Object) x);
```
👎 bad
```
foo (i, j);
args [0]; 
z = 2*x + 3*y;     
```

- class 정렬

필드(전역변수) -> 생성자 -> 메서드 순서로 정렬<br>

### Naming Rule


- 클래스와 인터페이스 명 : camel 표기법 사용

👍 good
```
XmlHttpRequest
long id 
```

👎 bad
```
getCustomerID
long ID 
```

- 패키지명 : 소문자만 사용해야하고 8자 이내, 복합단어의 사용 금지

ex) api, controller, service
ex) 
```
Examples:
    subwayzone  // 👎 bad
    zone.subway // 👍 good
```

### coding

- do..while의 사용 지양
- 메서드의 중간에 return 사용을 지양
- 증감 연산자는 분리된 라인에서 사용
- 변수 초기화는 사용되는 곳의 가까운 곳에 선언

