# ObjectFactory와 ObjectProvider

## singleton과 prototype을 같이 사용하는 경우의 문제점

- singleton으로 생성되는 clientbean
client에 의해서 prototype이 생성 된 이후에 관리된다. 이때 singleton(default)으로 clientbean을 생성하고 관리되는데 prototype의 특징이 유지될까?<br>
prototypebean을 clientBean에서 사용하기 위해서 의존성을 주입하는 과정을 거칠 것이다. 해당 단계에서 prototypeBean이 호출되게 되는데 이때 spring container에서 prototypeBean의 init이 이루어진다. 이렇게 되면 clientBean을 singleton으로 관리하는데 그 안에 prototypeBean이 주입되게 되는 것이다. <br>

따라서 ClientBean은 singleton이기에 prototypeBean은 호출될때마다 ClientBeand을 가져와서 prototypeBean을 사용하려는 경우에 다른 bean이 아닌 계속 같은 bean을 사용하게 된다.<br>

## 해결방법

그렇다면 새로운 clientBean을 사용할 때마다 prototypeBean을 새로운 spring container를 통해서 만들어줌으로써 다른 prototypeBean을 사용해도 되지 않을까? 라는 생각이 든다.<br>

그렇게 해도 된다. 하지만 계속해서 container를 생성하고 호출하게 된다면 무거운 spring container instance를 생성해줘야 할 것이다.<br>
다만 해당 아이디어는 가져간다! 이때 container를 새롭게 생성하고자 했던 것은 외부에 의해서 의존성을 주입해버리면 singleton으로 clientBean이 관리되기에 prototypeBean이 새롭게 생성되지 않아서 새로운 것을 만들어주려고 직접 새롭게 컨테이너를 생성함으로써 client에서 직접 필요한 의존관계를 찾아서 넣는 것을 의도했을 것이다. 이처럼 직접 필요한 의존관계를 찾는 과정인 Dependency Lookup을 진행한다는 아이디어만 가져가보자!<br>

### ObjectFactory , ObjectProvider

> DI(Dependency Lookup)은 의존관계를 외부에서 주입받는 것이 아니라 직접 의존관계를 조회하는 것을 의미한다.

이렇게 DI 기능을 수행하는 무언가가 ObjectFacotrym ObjectProvider이다.
<br>

- ObjectFactory , ObjectProvider

getObject method만 지원하지만 몇가지 편의기능을 추가한 것이 ObjectFactory이다.<br>
Spring Container를 통해서 bean을 찾아주는 기능만! 제공한다.<br>

    - spring이 제공하는 기능을 사용하지만 기능이 단순해서 단위테스트를 만들거나 mock 코드를 만들기에 쉽다.
    - spring이 제공하는 기능이기에 spring에 의존적이다.


그렇다면 spring에 덜 의존적인 방법은 없나? Java 표준을 사용한다.

### JSR-330 Provider

```
public interface Provider<T> {

    /**
     * Provides a fully-constructed and injected instance of {@code T}.
     *
     * @throws RuntimeException if the injector encounters an error while
     *  providing an instance. For example, if an injectable member on
     *  {@code T} throws an exception, the injector may wrap the exception
     *  and throw it to the caller of {@code get()}. Callers should not try
     *  to handle such exceptions as the behavior may vary across injector
     *  implementations and even different configurations of the same injector.
     */
    T get();
}
```
이렇게 등록되어있는 java 표준 라이브러리를 사용한다.

- 자바 표준이고 기능이 단순해서 단위테스트를 만들거나 mock 코드를 만들기가 훨씬 쉽다.
- DL 기능만 제공한다.