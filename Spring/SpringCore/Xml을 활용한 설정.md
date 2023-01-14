# Xml과 appConfig
-----
## xml을 활용한 appconfig 설정[what]

java code대신 xml문서를 활용하여 ApplicationContext에 AppConfig를 설정하는 방법이 존재한다.

-----

## 왜 알아야하나? [why]

- 레거시 프로젝트에서 XML로 되어있는 프로젝트가 많다는 점
- 컴파일 없이 빈 설정 정보를 변경할 수 있는 특징이 존재

-----

## 어떻게 사용해야하나? [how]

``` GenericXmlApplicationContext ```를 활용함.<br>

- appConfig.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">


    <bean id="memberService" class="tutorial.core.member.MemberServiceImpl">
        <constructor-arg name="memberRepository" ref="memberRepository" />

    </bean>
    <bean id="memberRepository" class="tutorial.core.member.MemoryMemberRepository"/>

</beans>
```
다음과 같이 작성이 가능하다.<br>
java, spring을 사용한 것과 비교를 해보자<br>

```
@Configuration
public class AppConfig {

    @Bean
    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }//생성자 주입

    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }
}
```
xml과 동일한 설정정보를 구성하고 있는 파일이다. 

- java code에서의 function name
    ApplicationContext에 등록되는 bean의 key값<br>
    xml에서는 id를 통해서 bean의 key값을 저장<br>
- java code에서의 return type
    ApplicationContext에 등록되는 bean의 value값<br>
    xml에서는 class를 통해서 bean의 value값을 저장<br>
- java code에서의 생성자주입부분
    의존성을 주입해주는 역할로 service에서 필요한 repository에 의존관계를 설정<br>
    constructor-arg를 통해서 의존관계를 설정하고 이때, 그 구현체를 ref를 통해서 설정<br>

- TestCode를 통한 사용방법 확인
    ~~~
    public class XmlAppContext {

        @Test
        void xmlAppContext(){

    
            ApplicationContext ac = new GenericXmlApplicationContext("appConfig.xml");
            //resources 밑의 appConfig.xml을 자동으로 읽어옴

            MemberService memberService = ac.getBean("memberService", MemberService.class);

            Assertions.assertThat(memberService).isInstanceOf(MemberService.class);
        }
    }
    ~~~
    
    위의 코드를 통해서 ``` GenericXmlApplicationContext ``` 를 통해 recources 하단에 저장해놓은 appConfig.xml이 읽히고 ApplicationContext에 등록해놓은 bean이 관리된다.<br>
    이때 ``` getBean ``` 을 통해서 applicationContext 즉, 스프링 컨테이너에 등록된 Bean을 id, type을 통해서 조회하고 MemberService에 담는다. <br>
    이때, 꺼낸 MemverService가 Bean 조회시 사용했던 type인 MemberService.class와 동일한지 체크하여 실제로 잘 등록되었는지 체크할 있다<br>

xml을 활용한 appConfig.xml작성 및 테스트코드 성공을 확인함으로써, 실제로 xml로 appConfig를 작성하는 방법과 사용하는 방법을 알아볼 수 있었다.