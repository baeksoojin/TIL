# BeanFactory와 ApplicationContext

BeanFactory와 ApplicationContext를 *스프링 컨테이너*라고 한다.

## BeanFactory

> 스프링 컨테이너의 최상위 인터페이스이다.

- 스프링 빈을 관리하고 조회하는 역할을 담당한다.
```
@Test
@DisplayName("타입으로만 조회")
void findBeanByType(){
    MemberService memberService = ac.getBean( MemberService.class);
    Assertions.assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
}
```
예를 들어서 스프링 빈을 조회하는 2가지 방법중 타입만을 사용하여 조회하는 코드를 작성할 때 다음과 같이 getBean을 이용한다. 이때 BeanFactory가 해당 기능을 제공한다.
<br>

## ApplicationContext

> 최상위 인터페이스를 상속받아서 만든 interface로 BeanFactory 기능에 부가 기능을 추가한 자식 인터페이스이다.

다시 위의 예제 코드를 보면 getBean의 메서드는 ApplicationContext에 의해서 호출되는 것을 알 수 있듯, BeanFactory가 제공하는 기능을 모두 상속받아서 제공한다.<br>

## BeanFactory와 ApplicationContext의 차이점

- ApplicationContext는 스프링 빈 조회 기능뿐만 아니라 다른 여러 기능을 포함한다.
1. interface MessageSource<br>
한국에서 들어온면 한국어로 영어권에서 들어오면 영어로 출력
2. EnvironmentCapable<br>
로컬, 운영, 개발 환경을 구분해서 처리하기 위한 환경변수 관련 기능제공
3. ApplicationEventPublisher<br>
이벤트를 발행하고 구독하는 모델을 편리하게 지원
4. ResourceLoader<br>
파일, 클래스패스, 외부 등에서 리소스를 편리하게 지원
5. BeanFactory<br>
빈을 관리 및 조회하는 기능

### 따라서 BeanFactory를 직접 사용할 일은 거의 없고 ApplicationContext를 사용한다.
