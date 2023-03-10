# DI

## DI란 무엇인가[what]

> Dependency Injection으로 run time에 외부에서 실제 구현 객체를 생성하고 클라이언트에 전달해서 클라이언트와 서버의 실제 의존관계가 연결되는 것을 의존관계 주입이라고 한다. 

- 정적인 class 의존관계

    application을 실행하지 않고도 의존관계를 판단할 수 있는 경우에 해당된다.

- 동적인 class 의존관계

    실제 어떠한 구현체가 어떤 것이 적용되는지는 모르기에 server를 켜봐야 알 수 있는 경우에 해당된다.

## 왜 사용하는가[why]

class diagram을 application을 개발할 때 미리 설계를 해놓는다. 이때 정적인 class diagram으로 불리는데 이는 구현체가 변경된다고 해서 계속 변화가 있어서는 안 된다. 이렇게 class diagram으로 정해진 전체적인 틀을 지키면서 개발을 해나갈 수 있도록 하려면 DI를 적절하게 사용해야한다.<br>

의존관계 주입을 사용하면 클라이언트 코드를 변경하지 않고, 클라이언트가 호출하는 대상의 타입 인스턴스를 변경한다<br>
정적인 클래스 의존관계를 변경하지 않고, *동적인* 클래스 의존관계만 쉽게 변경이 가능하다<br>

## 어떻게 사용하는가[how]

client의 코드를 변경하지 않고 제어권을 appconfig로 넘겨서 제어의 역전이 가능하다.<br>
참고)<br>
이때 AppConfig와 같이 객체를 생성하고 관리하며 의존관계를 연결해주고 주입시켜주는 것을 IoC container or **DI Container**라고 한다.<br>
