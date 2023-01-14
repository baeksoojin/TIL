# singleton 방식의 주의점


싱글톤 패턴 혹은 싱글톤 컨테이너 어느 것을 사용하던지 **객체 인스턴스 하나만 생성*해서 공유해서 사용하는 싱글톤 방식은 여러 클라이언트가 하나의 같은 인스턴스를 사용하기에 싱글톤 객체의 상태를 *유지해서는 안 된다*<br>

## 무상태 설계의 필요성


- 특정 클라이언트에 의존적인 필드가 있으면 안 된다.<br>
    = 클라이언트에서 값을 변경해서는 안 된다
- 가급적 값이 수정되지 않도록 읽기만 가능해야한다.

----

### Stateful하게 설계한다고 해보자

~~~
public class StatefulService {

    private int price;//상태를 유지하는 필드

    public void order(String name, int price){
        System.out.println("name="+name+"price"+price);
        this.price = price;
    }

    public int getPrice(){
        return price;
    }
}
~~~
다음과 같이 java class를 만들어놓는다. <br>

```
@Test
    void statefulServiceSingleton(){
        ApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);

        StatefulService statefulService1 = ac.getBean(StatefulService.class);
        StatefulService statefulService2 = ac.getBean(StatefulService.class);

        //threada : a사용자가 만원주문
        statefulService1.order("usera",10000);
        //threadb : b사용자가 2만원주문
        statefulService2.order("userb",20000);

        //threada : usera의 주문 금액 조호ㅚ
        int price = statefulService1.getPrice();
        System.out.println("price = "+price);

        org.assertj.core.api.Assertions.assertThat(statefulService1.getPrice()).isEqualTo(20000);

    }
```
결과가 어떻게 나올까?? 원하는 값은 10000원이겠지만 20000원이 나올 것이다<br>
singleton이 적용된다면 statefulService1과 statefulService2는 동일한 instance object이기때문이다. 따라서 처음에 10000원이였다가 20000원으로 price가 변경될 뿐만 아니라 name 역시 변경된다<br>

userA가 ThreadA를 호출, userB가 ThreadB를 호출한다고 해보자.<br>
두 Thread모두 같은 instance를 공유하고 있고 이때 order method를 통해서 update를 해준다. <br>
클라이언트에서 모두 값을 변경하고 있는데 *따라서 문제가 발생한다*<br>
Thread가 다르더라도 싱글톤 컨테이너를 사용하고 있기에 상태가 변화하게 되는 것이다.<br>

**만약에 결제 시스템에서 이렇게 된다면?? 큰일난다!!!!!!!<br>**

----

**그렇다면 어떻게 해야할까??**

- **자바에서 공유되지 않는 지역변수, 파라미터, ThreadLocal 등을 사용해야한다.**<br>

<br>

### stateless한 설계방법

```
public class StatelessService {

    private int price;//상태를 유지하는 필드

    public int order(String name, int price){
        System.out.println("name="+name+"price"+price);
        this.price = price;
        return price;
    }

    public int getPrice(){
        return price;
    }
}

```

다음과 같이 order에서 price를 return 해주고 이것을 사용하면 된다.<bt>

```
@Test
    void statelessServiceSingleton(){
        ApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig2.class);

        StatelessService statelessService1 = ac.getBean(StatelessService.class);
        StatelessService statelessService2 = ac.getBean(StatelessService.class);

        //threada : a사용자가 만원주문
        int userAPrice = statelessService1.order("usera",10000);
        //threadb : b사용자가 2만원주문
        int userBPrice = statelessService2.order("userb",20000);

        //threada : usera의 주문 금액 조호ㅚ
        int price = statelessService1.getPrice();
        System.out.println("price = "+price);

        org.assertj.core.api.Assertions.assertThat(userAPrice).isEqualTo(10000);

    }
```
return값을 "지역변수"로 받아와서 multi-thread 일때도 값이 사용자에게 알맞은 값이 찍히도록 해야한다.<br>

