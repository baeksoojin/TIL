# Springboot test와 @transaction annotation을 활용한 롤백

## @Transactional을 적용한 동작방식

Test가 아닌 곳에서의 Transactional 동작방식<br>

1. 트랜잭션의 시작<br>
2. logic 실행
3. rollback or commit을 진행

Test인 곳에서의 Transactional 동작방식<br>

1~2까지의 과정은 같아서 트랜잭션이 실행되고 test logic이 실행된다.<br>
하지만 3번 과정에서 rollback을 기본으로 가져간다.<br>
commit하지 않고 rollback해야하는 이유는 뒤에서 알아볼 것이다.<br>

다만, 기본이 rollback이고 commit을 사용하기 위해서는 강제로 커밋을 진행해야하는데, `@Commit`을 사용하면 된다.<br>
```
@Commit
@Transactional
@SpringbootTest
class className {}
```
따라서 위와 같이 적어주면 된다.<br>
참고로, `Rollback(value=false)`를 사용해도 된다.<br>

-----


## 그렇다면 Test에서는 왜 기본이 Rollback인지 알아보자.

그 이유는 MemoryDB가 아닌 다른 데이터베이스를 사용하는 경우, 그 데이터가 저장되고 없어지는 것이 아니라 계속 남아있기 때문이다.<br>
따라서 테스트의 **격리성**을 보장하기 위해서 rollback을 적용해야한다.<br>
남아있게 되면 문제점은, test를 분리하지 않게 되어 insert test 이후 findall test를 진행하게 되면 findall test에서 체크하기 위해서 넣은 데이터들이 잘 들어가는지 체크할 때, insert test에서 넣은 데이터도 같이 있어서 데이터가 오염된채로 테스트가 진행되어 정확한 테스트를 할 수 없다는데에 있다.<br>
따라서, test를 할 때마다 데이터를 초기화해줘야하는데 transactional과정에서는 rollback을 해주면 된다.<br>

참고로 쿼리를 통해서 clear해주는 방식은 clear해주기 전에 서버가 다운되어버리는 상황들에서 clear가 적용되지 않기 때문에 이후에 오염된 데이터베이스를 사용하여 테스트를 진행하게 된다.<br>
따라서, 이러한 문제가 발생하지 않도록 서버가 끊겨도 rollback이나 commit이 되지 않으면 해당 데이터가 업데이트 되지 않기에 데이터 초기화 방식으로는 rollback을 사용한다.<br>


------

참고로, test의 격리성을 보장하기 위해서, 개발과 TSET를 격리해야하는데 이때는 두개의 디비를 만들어서 격리시켜주는 방식을 적용해야한다.<br>


```
spring.profiles.active=local

#jdbcTemplate sql log
logging.level.org.springframework.jdbc=debug
spring.datasource.url=jdbc:h2:tcp://localhost/~/[local에서의 이름]
spring.datasource.username=sa
```
개발하는 환경의 profiles를 local로 적용하고 초기 데이터를 세팅하는 등 할 때 프로필을 활용한다.<br>
격리성을 위해서 datasource url을 local에서의 데이터베이스 url을 test환경에서의 url과 다르게 설정한다.<br>


```
spring.profiles.active=test

#jdbcTemplate sql log
logging.level.org.springframework.jdbc=debug
spring.datasource.url=jdbc:h2:tcp://localhost/~/[test에서의 이름]
spring.datasource.username=sa
```