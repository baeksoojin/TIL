## 양방향 의존관계의 주의사항

----


collection 조회가 아닌 다대일 일대일 양방향 의존관계에 의해서 참조가 일어나고 하나가 조회되는 경우에 발생가능한 문제들이 존재한다.<br>
이를 해결하고 성능최적화를 할 수 있는 방법에 대해서 알아보자.!<br>


------

> 두 객체가 서로 참조하는 관계이다.


양방향 의존관계에서의 주의점은 “순환참조”이다.

Order → Member → Order → Member → Order 등등

순환참조가 발생하게 된다.

따라서, 이를 해결하기 위해서 한쪽에서 순환참조를 하지 못하게 끊어줘야한다.

양방향이 걸리는 곳을 둘중 하나에서 `JsonIgnore`를 사용해서 무한루프를 끊어줘야한다.

## 지연로딩의 사용이유와 목적 및 주의사항

> 두 객체가 join되어있더라도 하나를 조회할 때, 다른 join된 테이블의 값은 바로 조회하지 않고 미루는 것을 의미한다.
> 

- 성능 개선을 위해 LAZY 활용

LAZY, EAGER 중에서 EAGER의 경우 (N+1)문제가 발생할 수 있다.

N+1 문제란 무엇인가? 나는 선생님한명과 관련 학생 한명을 조회하고 싶다고 해보자. 이때 선생과 학생 사이에 일대다 관계가 성립할 것이다. 만약 학생 1000명을 한명의 선생님이 관리할 때 우리가 알고싶은 학생은 한명인데 선생님을 조회할 때 연관된 학생들을 모두 조회하게 된다면 1001번의 쿼리를 날리게 된다. 이처럼 1개의 조회를 위해서 필요한 query가 N+1되 되는 문제를 N+1 문제라고 하고 EAGER일때 발생한다.

따라서, LAZY를 사용해서 연관된 table에 대한 query를 그때그때 바로 넘기지 않도록해야한다.

실문에서는 EAGER가 아닌 LAZY를 활용해야한다.

- 주의사항

다만, LAZY안에서 로직이 처리될 때, (N+1)의 문제가 역시 발생할 수 있다.

만약에 고객이 주문정보를 조회하고 싶어서 요청을 한 경우, 주문 Entity와 연관된 Entity가 Member와 Delivery라고 하자. 주문 정보를 조회하고 싶다는 request 하나에 대해 Order를 모두 조회하는 1번의 query를 가지고 총 2개의 주문 정보가 조회될 것이다.

이때, 2개의 Order에 맞게 2개의 Member, Delivery가 2번씩 조회되는 query가 안에서 LAZY가 초기화되면서 값을 불러올 것이다.

위의 상황에서 연관된 Entity가 2개여서 1+N+N의 문제가 발생하게 된다.

- Fetch Join의 사용

> fetch join은 한번의 쿼리로 필요한 것들을 다 불러오도록 join방법이다.
> 

EAGER는 사용하면 안 된다고 하고… LAZY를 사용해도 안의 로직에서 값을 불러오는 과정에서 LAZY가 초기화되며 1+N의 문제가 있다면 어떻게 해결할 수 있을까?

⇒ 해결방법은 `Fetch Join`을 사용하는 것이다.

⇒ join fetch 구문을 사용해야한다.

**Fetch Join과 영속성의 관계**

fetch join의 원리를 따져보면, 결국에는 **영속성을 언제 부여하냐?**의 문제이다.

fetch join은 LAZY가 초기화되면서 각각 영속성을 다른 시기에 부여하는 것이 아니라 한번에 영속성을 넣어줌으로써, 영속성 context에서 미리 모든 연관된 Entity를 관리하게 설정하여 이후 영속성을 넣어주지 않아서 다른 query가 넘겨지지 않게 한다.

**비교**

위의 주의사항에서 들었던 예시를 살펴보면, 1+N+N의 문제가 발생했던게 각 객체마다 영속성이 다른 시점에서 부여하게 돼서 commit시에 각각 sql query를 날리게 된다. 그래서 총 5번의 쿼리가 넘겨졌다.

영속성을 한번에 `fetch join` 을 사용해서 부여한다면 한번의 commit에 의해서 query하나가 넘겨지게 된다. 따라서 총 1번의 쿼리가 넘겨지도록 성능이 최적화되었다.

## network 사용량 줄이기

- 사용하고 싶은 데이터가 정해진 경우, fetch join으로 엄청 많은  select하도록 하지말고 사용하고 싶은 것만 DTO를 만들어서 처리한다.

다만, api spec에 맞춰서 query를 짠것이기에 논리적인 계층이 깨지게 만들 수 있기에 주의해야한다.

⇒ 성능 차이가 많이 나는가? 직접 test해보고 비교해보고 결정을 신중히 해야할 것이다.

- select보다는 join에서 큰 netwrok를 사용하게 된다.