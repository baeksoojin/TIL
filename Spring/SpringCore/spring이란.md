# Spring이란

## 스프링 생태계

필수
- [스프링 프레임워크](https://spring.io/projects/spring-framework)
- [스프링부트](https://spring.io/projects/spring-boot)

선택
- 스프링 데이터 : CRUD를 편리하게 제어할 수 있도록 도움(ex. spring data jpa)
- 스프링 세션 : session을 편리하게 사용
- 스프링 시큐리티 : 보안 관리
- 스프링 Rest Docs : api 문서화 편리하게함
- 스프링 배치 : 배치처리에 최적화된 기술
- 스프링 클라우드 : 클라우드를 적용

### Spring Framework

- **핵심기술** : DI 컨테이너, AOP(Aspect Oriented Programming), event, 기타
- 웹 기술 : spring MVC, spring webFlux
- 데이터 접근 기술 : transaction, JDBC, ORM, XML
- 기술통합 : cache, email, remoting, scheduling
- 테스트 : spring기반 테스트 지원
- 언어 : kotlin, groovy

### Spring Boot

Spring Framework을 기반으로 더 편리하게 애플리케이션을 생성하기 위해서 사용

- 독립형 Spring application 생성 <br>
build -> tomcat server -> project에 넣어서 띄우는 과정처럼 웹 서버를 내장해 별도의 웹 서버를 설치하지 않아도 됨
- 손쉬운 빌드 구성을 위한 starter 종속성 제공<br>
library를 가져다가 사용할 때 starter를 사용하면 필요한 나머지 library를 같이 당겨줘서 빌드 구성을 쉽게함
- 스프링과 3rd party 자동 구성 <br>
버전에 따른 외부 라이브러리 허용 범위를 직접 지정 등 체크
- 메트릭, 상태확인, 외부 구성 프로덕션 준비 기능 제공 <br>
운영환경에서 사용할 수 있도록 모니터링 기본을 제공해줌
- 간결한 설정방법<br>

----

그렇다면 왜 spring을 사용해야하고 왜 사용하고 있나? 왜 탄생하게 되었나?

## spring 탄생 배경

**핵심 컨셉**
- 자바 언어의 사용 - *객체 지향 언어*<br>
객체 지향 언어가 가진 강력한 특징을 살려내는 프레임워크<br>
EJB처럼 프레임워크에 의존하게 된다면 객체지향적인 특징을 살리지 못한다는 문제점에서 출발

**객체지향 프로그래밍**
>  객체 지향 프로그래밍은 컴퓨터 프로그램을 명령어의 목록으로 보는 시각에서 벗어나 여러 개의 독립된 단위, 즉 "객체"들의 모임으로 파악하고자 하는 것이다. 각각의 객체는 메시지를 주고받고, 데이터를 처리할 수 있다.

객체 지향 프로그래밍의 특징은 기본적으로 자료 추상화, 상속, 다형 개념, 동적 바인딩 등이 있으며 추가적으로 다중 상속 등의 특징이 존재한다.

- 다형성
> 어떤 한 요소에 여러 개념을 넣어 놓는 것. 역할과 구현의 분리가 명확하게 하기 위한 객체지향적 특징

**이해돕기!** 운전자 - 자동차<br>
운전자는 자동차의 구현이 바껴도(k3 -> 아반떼) 운전자는 운전이 가능하다.
운전자를 위해(client) 자동차의 구현이 바껴도 자동차를 운전할 수 있도록 자동차의 역할을 할 수 있게끔 만든다.
자동차의 역할을 바꾸지 않고(client에 영향을 주지 않고) 자동차를 무한히 확장이 가능하다.
=> 새로운 자동차가 나와도 client는 자동차를 다시 배우지 않아도됨

역할과 구현을 분리
- client는 대상의 역할(interface)만 알면 된다.<br>
- 클라이언트는 구현 대상(차 자체)의 내부 구조를 몰라도 된다.
- 역할 = 인터페이스(로미오 역할) / 구현 = 인터페이스를 구현한 클래스, 구현 객체(여배우 a, 여배우 b)

*다형성의 본질은 client를 변경하지 않고 server의 구현 기능을 유연하게 변경하게 함으로써 확장 가능한 설계를 가능하게 하는 것*

한계<br>
interface 자체가 변경되어야할 때, client와 server에 모두 큰 변경을 준다.<br>

*따라서 인터페이스를 안정적으로 잘 설계하는 것이 중요하다.*

**spring과 다형성**

- spring은 다형성을 극대화해서 사용하게끔 도와준다.
- IoC, DI는 다형성을 활용해서 역할과 구현을 편리하게 다룰 수 있도록 지원한다.


---

## 자바 언어의 다형성

- overriding을 활용<br>
부모 클래스로부터 상속받은 메서드의 내용을 변경
- interface를 통한 다중 상속<br>
자바는 클래스를 통한 다중상속을 제공하지 않기 때문에 interface를 사용한다.


    ~~~
    interface Animal { public abstract void cry(); }


    interface Cat extends Animal { public abstract void cry(); }

    interface Dog extends Animal { public abstract void cry(); }

    

    class MyPet implements Cat, Dog {

        public void cry() {

            System.out.println("멍멍! 냐옹냐옹!");

        }

    }

    public class Polymorphism05 {

        public static void main(String[] args) {

            MyPet p = new MyPet();

            p.cry();

        }

    } 
    ~~~
    결과는 멍멍! 냐옹냐옹! Cat 인터페이스와 Dog 인터페이스를 동시에 구현한 MyPet 클래스에서만 cry() 메소드를 정의하므로, 앞선 예제에서 발생한 메소드 호출의 모호성이 없다.


