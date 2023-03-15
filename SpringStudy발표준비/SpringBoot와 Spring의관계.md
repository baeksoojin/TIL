## SpringBoot 5가지 핵심기능

스프링을 더욱 편리하게 사용할 수 있도록한다.

- 내장 서버 : 별도의 서버 설치없이 손쉽게 개발 및 배포
- 자동 라이브러리 관리 : 라이브러리를 자동으로 관리
- 자동 구성 : 복잡한 스프링 설정을 자동화 ( Auto Configuration )
- 외부 설정 : 외부 설정값들을 편리하게 사용할 수 있도록 제공
- 모니터링 : 애플리케이션의 수많은 지표들을 조회하고 관리할 수 있도록함

## Springboot의 등장

- EJB를 사용하려면 수천만원짜리 소프트웨어를 사용했어야한다.
    - 다만 공부하기 복잡. EJB에 너무 의존적
    - POJO 순수한 자바를 사용해서 개발하자는 목소리
- Spring Framework가 등장
    
    spring framework를 기반으로하는 생태계가 커지고 늘어남
    
    spring이 무거워지기 시작.
    
    제대로 설정하기 위해서 spring을 이해하고 있어야함
    
- Spring 설정 지옥
    
    spring bean 등록 지옥
    
- Springboot 등장
    
    스프링을 편리하게 사용할 수 있도록 지원. 최근에는 기본으로 사용
    
    WAS : Tomcat server를 내장
    
    library 관리 : 관련된 library를 다 가져와줌,  library 버전을 자동으로 관리해줌
    
    Auto Config : 프로젝트 시작시 필요한 스프링과 외부 라이브러리 빈을 자동 등록
    
    외부설정 : 서버 환경이 달라지면 외부 설정을 읽어들여와야하는 상황이 생기는데 외부 설정을 공통화
    
    프로덕션 준비 : 모니터링을 위한 메트릭, 상태 확인 기능을 제공
    
    본질은 스프링 프레임워크.
    

---

## 1. Springboot의 Tomcat 내장

- tomcat server

![image](https://user-images.githubusercontent.com/74058047/225432345-2574ddf6-21e1-469b-b0ae-679931227e7c.png)


| 과거 | springboot |
| --- | --- |
| WAS를 따로 설치하고 서블릿 스펙에 맞춰서 코드를 작성하고 WAR 형식으로 빌드에서 war 파일을 만들었다. | WAR를 따로 설치할 필요없다. 내장 톰캣을 라이브러리로 제공해서 코드를 작성후에 JAR로 빌드하고 이를 원하는 위치에서 실행하기만 하면 된다. |
- jar란?

자바는 여러 클래스와 리소스를 묶어서 `JAR` (Java Archive)라는 압축파일을 생성할 수 있다.

이것은 JVM위에서 실행되고 그 위에 WAS가 동작한다.

- war란?

WAS 위에서 실행되고 WAS에 배포할 때 사용하는 파일이다. html같은 정적 리소스와 클래스 파일을 모두 함께 포함해서 JAR와 비교해서 구조가 더 복잡하다. 따라서 구조를 지켜야한다.

- WEB-INF
    - class, lib,web.xml
- index.html(WEB-INF가 아닌 영역)

### <과거>

### tomcat 띄우기

- setting

```java
plugins {
    id 'java'
    id 'war'
}
```

- code

```java
@WebServlet(urlPatterns = "/test")
public class TestServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("TestServlet.service");
        resp.getWriter().println("test");
    }
}
```

- 만약 실행하고 싶다면? → build

![image](https://user-images.githubusercontent.com/74058047/225432421-6b0d26bf-6dcd-4fd3-9df1-879a07e35e95.png)

![image](https://user-images.githubusercontent.com/74058047/225432450-e6064b7b-dc25-4f52-86fb-99ff1f458225.png)

war로 만들어지는 것을 알 수 있음

<구성>

`WEB-INF`안에 classes,lib등이 컴파일 돼 들어가있음

`index.html`

<tomcat에 배포>

![image](https://user-images.githubusercontent.com/74058047/225432481-6efd101d-8d6f-4989-b718-1e55b6caa695.png)

과정이 번거로운 것을 알 수 있음 → 따라서 **Intellij 로 편리하게 자동화**

- 설정과정

```java
task explodedWar(type: Copy) {
into "$buildDir/exploded"
with war }
```

![image](https://user-images.githubusercontent.com/74058047/225432609-ab70ecb0-1a0a-41fa-8f1b-61d9bfc812bd.png)
intellij setting → tomcat runner를 다운로드해서 설정.

### 실행

- 서블릿 컨테이너 초기화 방법 ex)

`ServletContainerInitializer` 초기화 인터페이스를 사용. 서블릿컨테이너를 초기화하는 기능을 제공하고 실행 시점에서 서블릿 컨테이너는 초기화 메서드인 onStartup()을 호출해서 서블릿을 등록한다.

```java
public class MyContainerInitV1 implements ServletContainerInitializer {
			@Override
      public void onStartup(Set<Class<?>> c, ServletContext ctx) throws
  ServletException {
          System.out.println("MyContainerInitV1.onStartup");
          System.out.println("MyContainerInitV1 c = " + c);
          System.out.println("MyContainerInitV1 ctx = " + ctx);
} }
```

`MyContainerInitV1` 해당 클래스가 초기화 클래스라는 것을 등록하기 위해서 `jakarta.servlet.ServletContainerInitializer` 경로안에 등록해줘야한다.

![image](https://user-images.githubusercontent.com/74058047/225432650-35c8c30f-1b60-479b-b1f6-354d94da6865.png)


주의)

file name, 경로 주의

위처럼 초기화 방법을 사용해서 servletcontainer를 등록. 등록된 파일에 접근해서 애플리케이션 초기화가 이루어짐.

### servlet을 만들어서 등록

- 서블릿 등록

```java
public class **HelloServlet** extends HttpServlet {
@Override
      protected void service(HttpServletRequest req, HttpServletResponse resp)
  throws ServletException, IOException {
          System.out.println("HelloServlet.service");
          resp.getWriter().println("hello servlet!");
      }
}
```

- 애플리케이션 초기화

```java
public interface AppInit {
      void onStartup(ServletContext servletContext);
}
```

`AppInit` 인터페이스는 필수! 

해당 인터페이스를 구현해서 실제 동작하는 코드를 작성가능

```java
public class **AppInitV1Servlet** **implements** **AppInit** {
      @Override
      public void onStartup(ServletContext servletContext) {
          System.out.println("AppInitV1Servlet.onStartup");
			//순수 서블릿 코드 등록 ServletRegistration.Dynamic helloServlet =
                servletContext.addServlet("helloServlet", new **HelloServlet**());
          helloServlet.addMapping("**/hello-servlet**");
} }
```

✍️ 프로그래밍 방식을 사용하고 @WebServelt annotation을 사용하지 않는 이유

```java
@WebServlet(urlPatterns = "/test")
```

해당 코드는 하드코딩되어있어서 경로 수정에 있어서 직접 코드를 찾아서 수정해야함.

하지만, 위의 코드에서의 appMapping안에 url은 외부에서 접근해서 수정이 가능함.

⇒ **유연성을 높이기 위해서 사용**
<br>
<br>


- interface 실행방법

```java
**@HandlesTypes(AppInit.class)**
public class MyContainerInitV2 implements ServletContainerInitializer {
			@Override
			public void onStartup(Set<Class<?>> c, ServletContext ctx) throws ServletException 
			{
          System.out.println("MyContainerInitV2.onStartup");
          System.out.println("MyContainerInitV2 c = " + c);
          System.out.println("MyContainerInitV2 container = " + ctx);
          
					for (Class<?> appInitClass : c) {
              try {
									//new AppInitV1Servlet()과 같은 코드
                  AppInit appInit = (AppInit)
								  appInitClass.getDeclaredConstructor().newInstance();
                  appInit.onStartup(ctx);
              } catch (Exception e) {
                  throw new RuntimeException(e);
         } //여러개가 꺼내질 수 있으니까 반복분.
} 
}
}
```

초기화 코드를 역시 지정 경로에 등록해줘야함. `MyContainerInitV2` 를 등록.

### 정리

![image](https://user-images.githubusercontent.com/74058047/225433071-dea097cf-ee4b-469f-8435-8486332657a3.png)

- 두가지과정을 분리한 이유

ServletContainerInitializer에 인터페이스를 구현한 코드를 만들어 넣어야 서블릿 컨테이너 초기화가 가능하고 이것을 또, 설정 파일 경로에 접근하여 직접 지정해줘야함.

하지만, 애플리케이션 초기화는 AppInit의 인터페이스만 구현하고 설정 파일 경로에 접근하여 직접 지정해줄 필요가 없음(처음한번빼고).

→ 편리함

애플리케이션 초기화는 서블릿 컨테이너와 상관없이 원하는 모양으로 인터페이스 생성이 가능.

→ 의존성분리

---

### 스프링 컨테이너 등록

![image](https://user-images.githubusercontent.com/74058047/225433154-7afca382-9788-4a12-bf8c-1d0bb9d9e24a.png)

- 설정

```java
dependencies {
//서블릿
implementation 'jakarta.servlet:jakarta.servlet-api:6.0.0'
//스프링 MVC 추가
implementation 'org.springframework:spring-webmvc:6.0.4'}
```

- code

controller

```java
@RestController
  public class HelloController {
      @GetMapping**("/hello-spring")**
      public String hello() {
          System.out.println("HelloController.hello");
          return "hello spring!";
      }
}

```

config

```java
@Configuration
public class HelloConfig {
    @Bean
    public HelloController helloController() {
        return new HelloController();
    }
 }
```

```java
public class AppInitV2Spring implements AppInit {
      @Override
      public void onStartup(ServletContext servletContext) {
          System.out.println("AppInitV2Spring.onStartup");

					//스프링 컨테이너 생성
          AnnotationConfigWebApplicationContext appContext = new
					  AnnotationConfigWebApplicationContext();
					appContext.register(HelloConfig.class); 

					//스프링 MVC 디스패처 서블릿 생성, 스프링 컨테이너 연결
          DispatcherServlet dispatcher = new DispatcherServlet(appContext);

					//디스패처 서블릿을 서블릿 컨테이너에 등록 (이름 주의! dispatcherV2) ServletRegistration.Dynamic servlet =
					servletContext.addServlet("dispatcherV2", dispatcher); 

					// /spring/* 요청이 디스패처 서블릿을 통하도록 설정
					servlet.addMapping("**/spring/*");**
      }
```

- 디스패처 서블릿이 http 요청이 들어오면 디스패처 서블릿은 해당 스프링 컨테이너에 들어있는 컨트롤러 빈을 호출.

---

### 스프링 MVC 서블릿 컨테이너 초기화 지원

![image](https://user-images.githubusercontent.com/74058047/225433210-4f8d27b7-1a86-47af-b455-0035b95730ea.png)

서블릿 컨테이너 초기화 → 애플리케이션 초기화 2가지 과정을 거쳤기 때문에 매우 복잡하다는 것을 확인가능.

spring은 앞의 단계인 서블릿 컨테이너를 초기화 하는 단계를 편리하게 지원한다.

- 서블릿 컨테이너 초기화

```java
public interface **WebApplicationInitializer** {
    void onStartup(ServletContext servletContext) throws ServletException;
}
```

`WebApplicationInitializer` 을 사용해서 서블릿 컨테이너 초기화를 편리하게 진행

spring-web library를 들어가보면, jakarta에 등록되어있음

WebApplicationInitializer를 보면, 앞서 만들었던 

```java
**@HandlesTypes(AppInit.class)**
public class MyContainerInitV2 implements ServletContainerInitializer {
			@Override
			public void onStartup(Set<Class<?>> c, ServletContext ctx) throws ServletException 
			{
          System.out.println("MyContainerInitV2.onStartup");
          System.out.println("MyContainerInitV2 c = " + c);
          System.out.println("MyContainerInitV2 container = " + ctx);
          
					for (Class<?> appInitClass : c) {
              try {
									//new AppInitV1Servlet()과 같은 코드
                  AppInit appInit = (AppInit)
								  appInitClass.getDeclaredConstructor().newInstance();
                  appInit.onStartup(ctx);
              } catch (Exception e) {
                  throw new RuntimeException(e);
         } //여러개가 꺼내질 수 있으니까 반복분.
} 
}
}
```

해당 로직을 처리해서 애플리케이션 초기화 구현체를 읽어들이는 작업을 진행하게 되는 것.

- 애플리케이션 초기화

```java
public class AppInitV3SpringMvc implements WebApplicationInitializer {
      @Override
      public void onStartup(ServletContext servletContext) throws
  ServletException {
          System.out.println("AppInitV3SpringMvc.onStartup");
					//스프링 컨테이너 생성
          AnnotationConfigWebApplicationContext appContext = new  AnnotationConfigWebApplicationContext();
					appContext.register(HelloConfig.class); //스프링 MVC 디스패처 서블릿 생성, 스프링 컨테이너 연결
          DispatcherServlet dispatcher = new DispatcherServlet(appContext);
					//디스패처 서블릿을 서블릿 컨테이너에 등록 (이름 주의! dispatcherV3) ServletRegistration.Dynamic servlet =
          servletContext.addServlet("**dispatcherV3**", dispatcher);
					//모든 요청이 디스패처 서블릿을 통하도록 설정
          servlet.addMapping("/");
      }
}
```

---

## springboot와 내장 톰켓

과거에는, 배포과정이 복잡하고 WAS를 설치해야한다는 점, 톰켓 버전을 변경할 때 톰켓을 재설치해야하는 것 등등의 단점이 존재.

어떻게 해결할까? → tomcat을 library로 제공하는 내장 톰켓을 제공한다.

JAR안에 library를 포함하는데 tomcat을 library로 내장. → main method를 실행할때 embedded방식으로 등록되어있어서 자동으로 실행됨.

### 내장 톰켓

- setting

plugin

```java
plugins {
         id 'java'
     }
```

dependencies

```java

implementation 'org.springframework:spring-webmvc:6.0.4'
//내장 톰캣 추가
implementation 'org.apache.tomcat.embed:**tomcat-embed-core**:10.1.5'

```

- servlet

```java
public class EmbedTomcatServletMain {
	public static void main(String[] args) throws LifecycleException { 
				System.out.println("EmbedTomcatServletMain.main");
								
				//톰캣 설정
				Tomcat tomcat = new Tomcat();
        Connector connector = new Connector();
        connector.setPort(8080);
        tomcat.setConnector(connector);

				//서블릿 등록
				Context context = tomcat.addContext("", "/");
				tomcat.**addServlet**("", "helloServlet", new **HelloServlet**());
        context.addServletMappingDecoded("/hello-servlet", "helloServlet");
        tomcat.start();
	}
}
```

→ **`HelloServlet` 이 servlet으로 등록되고 ‘**/hello-servlet’ 에서 실행된다.

- 내장 톰켓을 사용해서 톰켓을 설정하고 서블릿을 등록하여 사용할 수 있다는 것을 알면됨.

### 스프링 적용

```java
public static void main(String[] args) throws LifecycleException { System.out.println("EmbedTomcatSpringMain.main");
					//톰캣 설정
					Tomcat tomcat = new Tomcat();
          Connector connector = new Connector();
          connector.setPort(8080);
          tomcat.setConnector(connector);

					//스프링 컨테이너 생성
          AnnotationConfigWebApplicationContext appContext = new
						  AnnotationConfigWebApplicationContext();
					appContext.register(HelloConfig.class); 

					//스프링 MVC 디스패처 서블릿 생성, 스프링 컨테이너 연결
          DispatcherServlet dispatcher = new DispatcherServlet(appContext);
	
					//디스패처 서블릿 등록
					Context context = tomcat.addContext("", "/"); 
					tomcat.addServlet("", "dispatcher", dispatcher); 
					context.addServletMappingDecoded("/", "dispatcher");
          tomcat.start();
      }
```

### 빌드와 배포

`jar` 형식으로 빌드해야한다. 이때 실행할 `main()` method의 클래스를 지정해줘야한다.

- grdle을 통해서 클래스 설정

```java
Manifest-Version: 1.0
Main-Class: hello.embed.EmbedTomcatSpringMain
```

위처럼 세팅을 직접 해주는 것보다 Gradle의 도움을 받아서 쉽게 진행

```java
task buildJar(type: Jar) {
      manifest {
					attributes 'Main-Class': 'hello.embed.**EmbedTomcatSpringMain**'
					 }
			with jar 
}
```

- build file 생성

```java
./gradlew clean buildJar
```

→ libs에서 snapshot이 `.jar`로 생성된 것을 확인가능

- 실행

```java
java -jar [jarfile name]
```

- `Unable to initialize main class hello.embed.EmbedTomcatSpringMain`
    
    library가 없어서 오류발생.
    
    JAR 파일은 JAR파일을 포함할 수 없다.
    
    이때, `**FatJar` 를 활용할 수 있다.**
    

### FatJar

```java
task buildFatJar(type: Jar) {
      manifest {
          attributes 'Main-Class': 'hello.embed.EmbedTomcatSpringMain'
      }
      duplicatesStrategy = DuplicatesStrategy.WARN
      from { configurations.runtimeClasspath.collect { it.isDirectory() ? it :
  zipTree(it) } }
with jar }
```

- library안의 class를 다 뽑아서 넣어주는 코드

| 장점 | 단점 |
| --- | --- |
| 하나의 jar파일에 필요한 라이브러리를 내장할 수 있게되었다. 덕분에 하나의 jar파일로 웹, 서버 설치 실행 및 배포가 가능하다. | 용량이 매우 크다. 라이브러리를 풀어서 클래스를 넣는 것이여서 이후, 어떤 라이브러리가 포함되어있는지 확인하기 어렵다. 파일명 중복을 해결할 수 없다. META-INF/services/jakarta.servlet.ServletContainerInitializer 파일이 여러 library(jar)에 있을 때, 하나만 선택해서 실행되기때문에 결과적으로 나머지는 포함되지 않아서 정상 동작이 되지 않는다. |

![image](https://user-images.githubusercontent.com/74058047/225433302-2a9ac42b-de8c-4688-a726-8d614098bccb.png)

---

### 편리한 부트 클래스를 만들어보자

- MySpringApplication.run() 을 실행하면 기존 코드의 동작이 이루어지도록 설정

```java
public class **MySpringApplication** {
    public static void run(Class **configClass**, String[] args) {
        System.out.println("MySpringBootApplication.run args=" + List.of(args));

				 //톰캣 설정
        Tomcat tomcat = new Tomcat();
        Connector connector = new Connector();
        connector.setPort(8080);
        tomcat.setConnector(connector);

				//스프링 컨테이너 생성
        AnnotationConfigWebApplicationContext appContext = new AnnotationConfigWebApplicationContext();
				appContext.register(**configClass**);

				//스프링 MVC 디스패처 서블릿 생성, 스프링 컨테이너 연결
        DispatcherServlet dispatcher = new DispatcherServlet(appContext);

				//디스패처 서블릿 등록
				Context context = tomcat.addContext("", "/"); 
				tomcat.addServlet("", "dispatcher", dispatcher); 
				context.addServletMappingDecoded("/", "dispatcher"); try {
				tomcat.start();
				           } catch (LifecycleException e) {
				              throw new RuntimeException(e);
				} }
}
```

- annotation

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@**ComponentScan**
public @interface MySpringBootApplication {
}
```

- config

```java
//@Configuration
public class HelloConfig {
	       
	@Bean
	public HelloController helloController() {
	  return new HelloController();
	}
}
```

- main

```java
@MySpringBootApplication
  public class MySpringBootMain {
      public static void main(String[] args) {
          System.out.println("MySpringBootMain.main");
          **MySpringApplication**.**run**(**MySpringBootMain.class**, args);
} }
```

`MySpringBootApplication` annotation을 사용함과 동시에 패키지 하단의 모든 class에 대해서 component scan이 일어난다.

**MySpringBootMain.class가 곧 config class로 하단의 class를 찾아서 component scan이 발생하기 때문에 HelloConfig를 넣지 않고 MySpringBootMain.class를 넣어준다.**

---

## springboot와 웹 서버

```java
@SpringBootApplication
  public class BootApplication {
    public static void main(String[] args) {
        SpringApplication.run(BootApplication.class, args);
  
 } }
```

- 스프링 컨테이너를 생성
- WAS(tomcat)를 생성

### 실행

```java
java -jar boot-0.0.1-SNAPSHOT.jar
```

용량이 같은 로직을 작성한것인데 18M로 이전에 비해서 엄청 작아짐.

jar 압축파일을 풀면 ? → libs에 jar가존재. → **Jar안에 jar가 포함될 수 없는데 어떻게 가능했나??** `JarLauncher` 를 사용한다.

### 스프링 부트 실행 가능 Jar

jar 실행 코드가 입력되면, MANIFEST.MF를 인식하고 순차적으로 실행하는데, 가장 처음에 **Main-Class가 적혀있다.** `JarLauncher` 가 적혀있다.

어떤 라이브러리가 포함되어있는지 쉽게 확인이 가능하다.

jar를 실행시켜보면, **Main-Class**가 우리가 설정한 Main-Class가 아님을 확인 가능하다.

`JarLauncher` 라는 전혀 다른 클래스를 실행하는데, 스프링 부트가 빌드시에 넣어준다.

- JarLancher

JarLancher가 jar내부에 jar를 읽고 클래스 정보도 읽어온다. 

이후, 작업이 끝나면 → **Start-Class**로 우리가 지정한 main()을 호출해준다.



---

## 2. 스프링 부트의 라이브러리 직접 관리

### 라이브러리 관리의 어려움

어떤 라이브러리를 사용할지 고민하고 선택해야한다. 추가로 “버전”을 관리해줘야하는데 **호환이 잘 되는 버전이 있지만 잘 안 되는 버전이 존재할 수 있다.**

```java
dependencies {
//1. 라이브러리 직접 지정
//스프링 웹 MVC
implementation 'org.springframework:spring-webmvc:6.0.4'
//내장 톰캣
implementation 'org.apache.tomcat.embed:tomcat-embed-core:10.1.5'
//JSON 처리
implementation 'com.fasterxml.jackson.core:jackson-databind:2.14.1' //스프링 부트 관련
implementation 'org.springframework.boot:spring-boot:3.0.2' implementation 'org.springframework.boot:spring-boot-autoconfigure:3.0.2' //LOG 관련
implementation 'ch.qos.logback:logback-classic:1.4.5'
implementation 'org.apache.logging.log4j:log4j-to-slf4j:2.19.0' implementation 'org.slf4j:jul-to-slf4j:2.0.6'
//YML 관련
implementation 'org.yaml:snakeyaml:1.33'
}
```

### 스프링부트의 편리함

### 1. 버전관리

외부 라이브러리 버전을 관리해줄 뿐만 아니라 스프링 부트 스타터를 제공한다.

- version

```java
io.spring.dependency-management
```

plugin을 사용하여 버전을 등록하지 않아도 되고 **버전을 스프링부트가 관리해준다.**

- build.gradle 문서안에 보면 bom

> https://github.com/spring-projects/spring-boot/blob/main/spring-boot-project/spring-
boot-dependencies/build.gradle 안에서 bom확인이 가능하다.
> 

각각의 라이브러리에 대한 버전이 명시되어있어서 이를 활용해서 스프링 부트가 버전을 직접 관리해준다.

```java
plugins {
**id 'org.springframework.boot' version '3.0.2'**
id 'io.spring.dependency-management' version '1.1.0' //추가 id 'java'
}
```

**'org.springframework.boot' version '3.0.2’** 버전이 변경되면 나머지도 역시 변경되는 것을 확인이 가능하다.

- 스프링 부트가 관리하지 않는 라이브러리

> [https://docs.spring.io/spring-boot/docs/current/reference/html/dependency-versions.html#appendix.dependency-versions.coordinates](https://docs.spring.io/spring-boot/docs/current/reference/html/dependency-versions.html#appendix.dependency-versions.coordinates) 를 통해서 외부 라이브러리 버전을 확인이 가능하다.
> 

잘 알려지지 않은 경우, 등록되어있지 않을 수도 있어서 라이브러리 버전을 직접 넣어줘야한다.

### 2. 라이브러리 통합관리

```java
implementation 'org.springframework.boot:**spring-boot-starter-web'**
```

웹 프로젝트 안의 필요한 라이브러리를 모두 뫃아놓은 스프링 부트 스타터를 사용하면 된다.

| before | after |
| --- | --- |
| dependencies {모든것을 직접 세팅해줘야함} | dependencies { implementation 'org.springframework.boot:spring-boot-starter-web'} |

- 엄청많은 스프링 스타터가 존재한다.

> [https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.build-systems.starters](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.build-systems.starters) 해당 링크를 통해서 스프링 스타터를 찾아서 사용하면 된다.
> 

사실 프로젝트를 만드는 [https://start.spring.io/](https://start.spring.io/) 에서 필요한 것을 등록해주면 되긴하다.

### 3. 예외

외부 라이브러리의 버전을 변경하고 싶다면, build.gradle에서 `ext['tomcat.version'] = '10.1.4’` 와 같이 버전정보를 넣어줌으로써 간편하게 변경이 가능하다.

하지만, 스프링 부트가 관리하는 외부 라이브러리 버전을 직접 변경하는 일은 거의없다. 하지만 가끔 문제가 발생할 수 있는데 이때는 버전 변경에 필요한 속성 값을 찾아보며 변경하면 된다.

> [https://docs.spring.io/spring-boot/docs/current/reference/html/dependency-versions.html#appendix.dependency-versions.properties](https://docs.spring.io/spring-boot/docs/current/reference/html/dependency-versions.html#appendix.dependency-versions.properties) 여기에 들어가서 ‘tomcat.version’과 같은 표현식을 찾아주면 되고 그것을 사용해, 버전을 변경하면 된다.
>