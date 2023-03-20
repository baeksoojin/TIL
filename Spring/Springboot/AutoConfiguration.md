# AutoConfigration

## springboot이 제공하는 수 많은 자동 구성

`spring-boot-autoconfigure`에 자동 구성을 모아놓는데, 스프링 부트 프로젝트를 사용하게 되면 `spring-boot-autoconfigure` library가 기본으로 사용된다.
<br>

따라서, springboot 기반의 프로젝트를 사용할 때는 spring에서 이미 자동으로 구성되도록 설정해놓은 `JdbcTemplate` 혹은 `DataSource` 혹은 `TransactionManager` 등을 직접 bean으로 등록하지 않아도 사용이 가능한 것이다.<br>

스프링은 위의 세가지뿐만 아니라 수많은 자동 구성을 모아놨기 때문에 boot를 사용하지 않았을 때보다 더 편리하게 configuration이 가능하다.

## springboot가 가능하게 해주는 직접 만든 library의 자동구성

스프링부프 프로젝트를 띄울 때 `ImportSelector` 의 구현체인 `AutoConfigurationImportSelector` 가 동작한다.<br>
`AutoConfigurationImportSelector`의 동작을 이해함으로써 library 자동구성에 대해서 알아볼 수 있다.<br>

- `AutoConfigurationImportSelector` 의 동작

`META-INF/spring/org.springframework.boot.autorconfigure.AutoConfiguration.imports` 의 경로에서 등록된 것들을 읽는다.<br>

### 그렇다면 해당 경로에는 어떤 것이 등록이 되어있나?

직접 library를 만들 때, 자동구성 class를 만드는데 이때 `AutoConfiguration` annotation을 활용해서 만든 부트가 뜰 때 자동으로 bean이 등록될 수 있도록 한다. 이때 만든 AutoConfiguration class file을 `META-INF/spring/org.springframework.boot.autorconfigure.AutoConfiguration.imports` 경로 안에 pakage 명을 포함해서 등록해놓는다. <br>
이것은 다른 프로젝트에서 build한 library의 jar 파일만 gradle에 등록하기만 하면 직접 필요한 bean을 등록하지 않아도 되도록해준다.<br>

따라서, 스프링부트는 동작할때 `AutoConfigurationImportSelector` 가 실행되게끔 하기 때문에 library에서 필요한 bean들을 자동으로 등록해주고 gradle에 등록만 해주면 그 기능을 사용할 수 있는 것이다.

## 자동 구성의 사용

라이브러리를 만들어서 제공할 때 사용할 수 있다.<br>
사실 이것보다 중요한 것은 스프링 부트의 자동 구성 코드를 읽어서 특정 빈들이 어떻게 등록되어 동작하는 것인지 이해할 필요가 있다.<br>
문제가 발생했을 때 대처가 가능하도록 하기 위해서는 자동 구성이 어떻게 제공되는지에 대한 이해와 자동구성 코드를 읽을 수 있어야한다.<br>
