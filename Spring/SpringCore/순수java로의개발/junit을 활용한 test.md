# JUnit을 활용한 Test Code 작성

## JUnit이란
> JUnit이란 Java 및 JVM을 위한 프로그래머 친화적인 테스트 프레임워크이다.

Junit5는 이전 버전과 달리 세가지 하위 프로젝트의 여러 모듈로 구성된다.

1. JUnit paltform<br>
JVM에서 테스트 프레임워크를 시작하기 위한 base 역할을 한다. TestEngine 플랫폼에서 실행되는 테스트 프레임워크를 개발하기 위한 API를 정의한다.<br>
인기있는 JUnit platform 지원 IDE는 Intellij, Eclipse, NetBeans 등이 있다.
2. JUnit Jupite<br>
JUnit5에서 테스트 및 확장을 작성하기 위한 프로그래밍 모델과 확장 모델의 조합이다.<br>
Jupiter 하위 프로젝트는 TestEngine 플랫폼에서 Jupiter 기반 테스를 실행하기 위해서 제공된다.
3. Junit Vintage<br>
TestEngine은 플랫폼에서 JUnit3 JUnit4 기반 테스트를 실행하기 위해 제공됩니다.<br>
즉, 하위호환성을 제공하기 위해서 하위버전의 TestEngine을 제공해줍니다.

## Junit 사용이유[why]

단위 테스트를 진행할 때 application의 psvm(public static void main)을 통해서 testcode를 작성한다고 해보자.
```

import hello.core.member.Grade;
import hello.core.member.Member;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;

public class MemberApp {

    public static void main(String[] args) {
        MemberService memberService = new MemberServiceImpl();
        Member member = new Member(1L, "member1", Grade.VIP);
        memberService.join(member);

        Member findMember = memberService.findMember(1L);
        System.out.println("new member = " + member.getName());
        System.out.println("find member = "+findMember.getName());
    }
}
```
**System.out.println 등 로그를 찍어가면서 test 결과를 체크해야한다는 불편함이 존재한다.**<br>
직접 테스트 결과를 체크하기에는 서비스가 복잡해질수록 프로그래머 입장에서 불편한 일이다.

따라서, 프로그래머 친화적인 Test Framework Junit을 사용한다.
```
public class MemberServiceTest {

    MemberService memberService = new MemberServiceImpl();
    @Test
    void join(){
        //given
        Member member = new Member(1l,"memberA", Grade.VIP);

        //when
        memberService.join(member);
        Member findMember = memberService.findMember(1l);

        //then
        Assertions.assertEquals(member, findMember);

    }
}

```
다음과 같이 Intellij에서 JUnit TestEngine을 사용해서 테스트가 가능하다.

## 사용방법[how]

- testcase example
    ~~~
    import static org.junit.jupiter.api.Assertions.assertEquals;

    import example.util.Calculator;

    import org.junit.jupiter.api.Test;

    class MyFirstJUnitJupiterTests {

        private final Calculator calculator = new Calculator();

        @Test
        void addition() {
            assertEquals(2, calculator.add(1, 1));
        }

    }
    ~~~

- Annotation


    |annotation|description|
    |------|------|
    |@Test|메서드가 test 메서드임을 알려줍니다. |
    |@ParameterizedTest| 메서드가에 매개변수를 넘겨 테스트를 함을 나타냅니다. @ValueSource annotation을 사용해서 매개변수를 넘겨 메서드를 수행하고 테스트 이후 개별적으로 결과가 보고됩니다. |
    |@ExtendWith|test 메서드 정의시 하위 클래스에 특정 테스트 메서드를 extension합니다. 여러가지 test method를 extention 적용이 가능합니다.|
    |@BeforeEach|해당 메서드는 다른 메서드(@Test @RepeatedTest @ParameterizedTest @TestFactory 등)들의 실행 전에 먼저 실행이 됩니다. 따라서 각 메서드들의 테스트 이전에 필요한 세팅을 미리 해줘야할때 사용합니다.|
    |@AfterEach|해당 메서드는 다른 메서드(@Test @RepeatedTest @ParameterizedTest @TestFactory 등)들의 다음에 실행되어야 합니다.|
    |@BeforeAll|모든 메서드보다 해당 메서드가 먼저 실행되어야 합니다. 테스트가 실행되기 전 딱 한번만 메서드가 실행됩니다. |
    |@AfterAll|모든 메서드보다 해당 메서드가 이후에 실행되어야 합니다. 테스트가 실행된 후 딱 한번만 메서드가 실행됩니다.|

    Jupiter Concept
    1. Lifecycle Method
    @BeforeAll, @AfterAll, @BeforeEach, or @AfterEach 를 활용한다.
    2. Test Class
    적어도 하나의 test method를 포함해야한다. Test class는 반드시 추상 클래스가 아니여야하고 단일 생성자가 존재해야한다.
    3. Test Method
    @Test, @RepeatedTest, @ParameterizedTest, @TestFactory, or @TestTemplat 혹은 directly annotated된 메서드입니다. @Test를 제외한 메서드들은 group test, ohter containers의 test tree에 컨테이너(e.g. a test class)가 생성됩니다.
    <br>

    이외에도 다양한 Annotation이 존재하고 https://junit.org/junit5/docs/current/user-guide/#writing-tests 해당 링크를 참고해서 사용하면 될 것이다.

그렇다면 spring project를 진행할때는 JUnit을 어떻게 사용하면 될까? 라는 생각이 들 수 있다.

## SpringBoot와 Test

Spring Boot에서는 Bean의 규모를 제어하며 테스트를 진행할 수 있는데 각각 annotation을 통해서 테스트 단위를 알려 줄 수 있다.<br>
Spring Boot에서 사용하는 Test annotation에 대해서 이후에 알아볼 것이다!
