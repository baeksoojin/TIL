## spring이 제공하는 프록시기술 - 프록시팩토리

일관성있고 편리하게 사용할 수 있는 프록시 팩토리라는 기능을 제공한다.

> 프록시 팩토리란, 스프링이 편리하게 동적 프록시를 만들 수 있도록 하는 것이다.

<img width="708" alt="image" src="https://user-images.githubusercontent.com/74058047/224549850-00eafcfa-ddae-4496-b583-bdd9afc838db.png">


- interface가 있는지 구체class만 존재하는지를 통해서 어떤 방식으로 프록시를 만드는지 개발자가 관여하지 않는다. → ProxyFactory

따라서, 개발자는 `InvocationHandler` 혹은 `MethodInterceptor` 를 몰라도 된다.

 `Advice` 를 통해서 해결이가능하다.

프록시 팩토리는 내부에서 프록시를 만들고 그 결과에 의해서 `adviceInvocationHandler`가 만들어지거나 `adviceMethodInterecptor`가 만들어진다.

이때, 그 두개의 handler는 모두 `Advice`를 호출한다. 이를 정의함으로써 둘중에 어떤건지 개발자는 고려하지 않고 통일성있게 proxy사용이 가능하다.

- proxy 생성 → ProxyFactory
- logic → Advice
- 특정 조건에 맞을 때만 프록시 로직을 도입하고 싶을 때 → PointCut