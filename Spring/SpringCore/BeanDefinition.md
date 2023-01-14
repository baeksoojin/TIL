# BeanDefinition

## BeanDefinition이란
> spring이 java, xml... 등등의 다양한 설정 형식을 지원할 수 있도록 추상화를 사용하는데 이를 BeanDefinition이다.

- config를 위한 역할과 구현중 "역할"이 BeanDefinition이라고 보면 됨.<br>
    "구현"은 그렇다면 AppConfig.class or appConfig.xml 등등이 되겠음<br>
- spring container(ac)는 설정정보의 소스코드가 어떤 것이냐에 영향을 받지 않음<br>
    "구현"인 AppConfig.class or appConfig.xml 등등에 따라서 달라지지 않고 오로지 ``` beanDefinition ```만 알고 동작하게 됨<br>

` BeanDefinition `란? **빈 설정 메타 정보**<br>
    - bean 하나당 각각의 메타정보가 `@Bean` or ` <bean> `으로 메타정보가 생성됨.
    - spring container(ac)는 해당 메타정보를 보고 스프링 빈을 생성함.


```
public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement { }
```
BeanDefinition을 찾아보면 위에서 볼 수 있듯이 "추상화"를 활용한 **interface**임.


### Bean등록과정을 세부적으로 살펴보자!
` AnnotationConfigApplicationContext `와 BeanDefinition의 관계
```
public class AnnotationConfigApplicationContext extends GenericApplicationContext implements AnnotationConfigRegistry {

	private final AnnotatedBeanDefinitionReader reader;
    //여기서 보면 AnnotatedBeanDefinitionReader가 있고 해당 reader기를 통해서 appconfig를 읽고 BeanDefiniton이 메타정보를 생성할 수 있도록 만들어주는 역할을 함

	private final ClassPathBeanDefinitionScanner scanner;
```
- AnnotatedBeanDefinitionReader
java code를 읽어서 BeanDefinition을 생성한다

` GenericXmlApplicationContext `와 BeanDefinition의 관계
```
public class GenericXmlApplicationContext extends GenericApplicationContext {

	private final XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(this);

 //생략
}
```
- XmlBeanDefinitionReader 를 통해서 xml을 읽어서 BeanDefinition을 생성한다.

----

## BeanDefinition의 구성정보

- BeanClassName : 생성한 빈 클래스 명(자바 설정은 팩토리 역할의 빈을 사용하여 없음)
- factoryBeanName : 팩토리 역할의 빈을 사용할때의 이름<br>
    ex)factoryBeanName=appConfig
- factoryMethodName : 팩토리 역할 빈 생성시시의 팩토리 메서드 지정 이름<br>
    factoryMethodName=orderService;
- Scope : 싱글톤(없다면 싱글톤)
- lazyInit : 빈의 생성을 지연하여 생성하는지에 대한 여부
<생명주기 관련>
- InitMethodName : 빈을 생성하고 의존관계를 적용한 뒤 호출되는 초기화 메서드 명
- DestoryMethodName : 빈의 생명주기가 끝나서 제거하기 직전에 호출되는 메서드 명
- Constructor arguments, Properties : 의존관계 주입시 사용되지만 자바처럼 팩토리 역할의 빈을 사용하면 없음.

BeanDefinition은 직접 프로그래밍하여 Bean을 등록할 수 있지만 사실 이럴일은 거의없다.<br>
하지만 다른 코드를 읽을 때 필요할 수 있으니 **BeanDefinition이 구성파일의 빈을 읽어온 정보를 기반으로 메타정보가 생성된다는 것에대한** 원리를 이해하는 것이 필요하다.

----
springBean은 크게 factoryBean을 사용하여 등록하는 방법(java code)과 직접등록(xml)을 통해서 등록하는 방법이 
존재한다<br>

- java code를 통한 구성<br>
    FactoryBean에 의해서 등록되고 관리되기 때문에 class 정보가 비어있고(null) factoryBeanName factoryMethodName 존재한다.
- xml을 통한 구성
    class는 xml 파일에서 등록한 class로 등록되고 factoryBeanName와 factoryMethodName가 비어있다.


