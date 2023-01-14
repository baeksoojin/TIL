# web application과 singleton

------

## singleton pattern [what]

> 객체를 하나 생성하여 공유해서 사용함으로써, 요청의 갯수에 영향을 받지 않는 패턴이다.

- class의 인스턴스가 1개만 생성됨을 보장(절대 2개이상 생성되지 않도록함)하는 디자인 패턴<br>
    **private**으로 막아서 new를 통해서 새로운 object instance가 생성되지 않도록한다.
    new로 객체 생성을 시도할경우, **컴파일오류를 유도하도록 설계**한다.


- 장점
    메모리 낭비를 줄이고 엄청난 트래픽에 대비가 가능하다.

- 단점
    DIP를 위반한다. -> appConfig에서 구체 클래스에 의존하게 된다.<br>
    미리 instance를 받아서 설정이 끝나있는 패턴이기에, Test의 유연성이 떨어질 수 있다.<br>
    private 생성자를 사용해서 자식 클래스를 만들기 어렵다.<br>
    클라이언트가 구현체에 의존해서 OCP를 위반할 가능성이 높다.<br>

단점이 많다.... 메모리를 줄이기 위해서 singleton을 사용해야 할 것 같은데 단점이 많으면 어쩌지?라는 생각이 들 것이다.<br>
여기서 Spring을 사용해야하는 또 다른 이유를 찾을 수 있다<br>

Spring은 DIP와 OCP를 어기는 막아주고 위의 단점을 모두 극복한 Spring Container를 제공한다.

----

## singleton pattern의 사용이유 [why]

- 만약 고객요청이 많을때, 그때마다 객체가 새롭게 생성되어야한다면 메모리 낭비가 매우 심하다는 단점이 존재한다.<br>
    따라서 singleton pattern을 사용한다.

- 메모리를 줄이기 위해서 사용한다.
- 트래픽이 많을 경우, 동시요청에 효율적인 메모리를 가져간다.

-----

## singleton pattern [how]


### spring 없는 순수한 DI container
- 구현하는 방법은 다양하다.
- 객체를 미리 생성해두는 가장 단순하고 안전한 방법

~~~

public class SingletonService {

    private static final SingletonService instance = new SingletonService();//생성

    public static SingletonService getInstance(){//조회
        return instance;//항상 같은 Instacne를 반환.(1개의 객체 인스턴스만 존재할 수 있기 때문이다)
    }


    private SingletonService(){

    }

    public void logic(){
        System.out.println("singleton object 로직 호출");
    }
}

~~~

다음과 같이 스프링없이 singleton pattern을 작성할 수 있다.<br>
위의 코드는 static 변수를 활용해 instance 오브젝트를 한번만 할당해준다.<br>
외부에서 사용할때는 SingletonService.getInstance()를 통해서 등록된 instance를 꺼내올 수 밖에없다<br>
만약 new 키워드를 통해서 update하게 된다면 위의 과정이 의미가 없을 것이기에 *생성자를 private*으로 설정한다<br>


### spring container와 singleton [중요]

```
@Test
    @DisplayName("스프링 컨테이너와 싱글톤")
    void springContainer(){

        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

        //1. 조회 : 호출시마다 객체 생성하는지 조회
        MemberService memberService1 = ac.getBean("memberService", MemberService.class);

        //2.  조회 : 호출시마다 객체 생성하는지 조회
        MemberService memberService2 = ac.getBean("memberService", MemberService.class);


        //참조값이 다른지 확인 -> 새로운 객체가 계속 생성됨을 알 수 있음.
        System.out.println("memberService1"+memberService1);
        System.out.println("memberService2"+memberService2);

        //자동화 -> 서로 다른지 확인
        Assertions.assertThat(memberService1).isSameAs(memberService2);

    }
```
spring을 통해 Bean을 등록한 후에 test를 하는 코드이다.<br>
잘 돌아가는 것을 통해서 동일한 MemberService를 요청마다 반환한다는 것을 알 수 있다.<br>



- spring container는 singleton container의 역할을 한다.
    싱글톤 객체를 생성하고 관리하는 기능을 싱글톤 레지스트라고한다.<br>
    spring container는 beanDefinition을 통해서 container에 먼저 name,value를 등록한다. 이후에 등록되어있는 똑같은 object를 호출해서 사용한다. 이것처럼 spring container는 singleton pattern 기능을 제공한다. 이것을 **싱글톤 레지스트리**라고 한다.<br>
- sinpleton  pattern을 위한 위의 코드처럼 직접 작성하지 않아도 된다.

