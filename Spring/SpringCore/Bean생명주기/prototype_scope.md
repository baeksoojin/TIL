# prototye scope

## prototype과 Bean의 관계

<container의 프로토타입 빈 생성>
- spring container가 bean을 생성하는 시점
client의 요청이 있을 때, 스프링컨테이너는 그 시점에 prototype Bean을 생성한다.<br> 이후 필요한 의존관계를 주입하여 Bean을 세팅한다<bt>

- spring container가 bean을 반환하는 시점
client에게 생성한 prototype bean을 반환하고 관리를 하지 않는다.<br>
같은 요청이 오면 **항상 새로운 프로토타입 빈**을 생성해서 준다.<br>

### 스프링 컨테이너는 프로토타입 빈을 생성하고 , 의존관계를 주입한 다음, 초기화하는 것까지만 처리한다는 점이 중요하다.

따라서 프로토타입 빈을 관리하는 책임은 프로토타입 빈을 받은 클라이언트에 있기에 `@PreDestory` 같은 종료 메서드 자체가 호출되지 않는다.<br>

## prototype 적용방법

- annotation을 활용

`@Scope("prototype")`을 spring bean을 생성할 메서드 위에 적어준다.<br>
destory는 관리하지 않기에(강제로 종료할 수는 있음) init을 위해서 `@PostConstruct`를 통해서 bean이 호출될때 spring container에 prototype bean생성 이후 의존관계 주입 그 다음으로 init을 진행한다.<br>

- 예제
~~~

public class PrototypeTest {

    @Test
    void prototypeBeanFind(){
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(PrototypeBean.class);

        System.out.println("find prototypeBean1");
        PrototypeBean prototypeBean1 = ac.getBean(PrototypeBean.class);
        System.out.println("find prototypeBean2");
        PrototypeBean prototypeBean2 = ac.getBean(PrototypeBean.class);
        System.out.println("prototypeBean1 "+prototypeBean1);
        System.out.println("prototypeBean2 "+prototypeBean2);

        //destroy가 호출되지 않는다.
        // 호출할 필요가 있다면 .destroy로 직접 종료 메서드사용

        Assertions.assertThat(prototypeBean1).isNotSameAs(prototypeBean2);
        ac.close();
    }

    @Scope("prototype")
    static class PrototypeBean{
        @PostConstruct
        public void init(){
            System.out.println("PrototypeBean.init");
        }

        @PreDestroy
        public void destroy(){
            System.out.println("PrototypeBean.close");
        }
    }


}

~~~
결과는 어떻게 될까?<br>
Test는 성공한다. <br>
그렇다는 것은 PrototypeBean1과 PrototypeBean2가 다르다는 것을 의미한다. 이처럼 prototypeBean의 경우에는 spring container에서 client에게 반환하고 더이상 관리하지 않음을 의미한다.<br>
그렇기때문에 당연히 destory역시 ac.close() 명령을 해도 종료직전에 @PreDestory는 먹히지 않아서 PrototypeBean.close를 반환하지 않는다.<br>

### singleton과의 차이점

위에서는 singleton과의 차이점을 역시 확인할 수 있다.<br>
우선 singleton은 spring container가 생성되는 시점에 springBean을 등록하고 DI를 설정한 후에 초기화를 진행하고(외부와의 연결 등) 이후에 사용이 되게끔 관리된다.<br>
하지만 prototypeBean의 경우 container가 생성되는 시점에 Bean을 만드는 것이 아니라 Bean이 호출되는 시점에 생성된다는 것을 알 수 있다.<br>
    ~~~
    find prototypeBean1
    PrototypeBean.init
    find prototypeBean2
    PrototypeBean.init
    ~~~
    그렇기에 결과가 다음과 같이 출력되는 것을 확인 할 수 있다.

물론 위에서 언급한 것처럼 초기화하여 반환한 이후에 관리를 해주는가에 대한 여부에서도의 차이점이 존재한다.<br>

