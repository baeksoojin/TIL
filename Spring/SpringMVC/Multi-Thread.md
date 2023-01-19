# Muli-Thread

------

## Multi-Thread의 개념

- WAS에서 Thread의 동작
WAS에서 Servlet을 호출하는데 Thread가 해당 역할을 해준다.<br>

- 단일 요청일때
Thread는 조건변수를 사용해서 요청을 체크하게 되는데 요청이 들어오면 servlet에서 처리를 진행한다.<br>

- 다중 요청일때
Thread를 하나만 사용하게 된다면 먼저 들어온 요청을 사용할 때 lock 상태이기에 다음에 들어온 요청을 처리하지 못하게 되고 timeout이 될 수 있다.<br>
따라서 servlet 객체의 처리에 연결을 해주는 Thread를 여러개 생성하면 된다.<br>

<요청이 올 때마다 Thread를 생성하게 될때>
Thread의 생성과정에서의 절차와 사용절차시 비용이 매우 크다.<br>
Thread는 switching(core 하나가 Thread를 동시에 사용하는것처럼 빠르게 Thread를 전환해주는 것) 비용이 발생한다.<br>
고객 요청이 너무 많으면 Thread를 계속생성해야할때 서버가 다운될 수 있다.<br>

------

## ThreadPool
> Thread를 먼저 많이 만들어놓고 필요할때마다 사용할 수 있다.

- 장점
요청이 많아서 Threadpool에 Thread가 존재하지 않을때 대기하고 거절하는 역할을 제공할 수 있다.<br>
생성 가능한 최대치가 있어서 너무 많은 요청이 들어와도 기존 요청은 안전히 처리가 가능하다.<br>

- 사용
쓰레드가 필요하면 이미 생성된 쓰레드를 풀에서 꺼내서 사용한다.<br>
사용한 이후에는 다시 쓰레드 풀로 반납한다.<br>
traffic이 몰릴때 pool안에 있는 Thread가 다 꺼내져있을 때 거절하거나 대기하도록 설정이 가능하다.<br>

- Max Thread 수
주요 튜닝 포인트이다.<br>
동시 요청이 많을 때 너무 낮게 설정한다면 CPU 사용률이 남아도는데 몇사람의 요청밖에 사용하지 못한다.<br>
동시 요청이 많을 때 너무 많이 설정한다면 CPU,Memory resource 임계점 초과로 서버가 다운된다.<br>

성능 테스트
application logic의 복잡도, cpu, memory, IO resource 상황에 따라서 모두 다르기에 "성능 테스트"를 진행해야한다.<br>
- 최대한 실제 서비스와 유사하게 성능 테스트 시도
- 툴 : 아파치 ,제이미터, nGrinder를 사용한다.<br>

------

## WAS의 멀티쓰레드 지원

- Multi-Thread 관련 코드를 신경쓰지 않아도 되기에 개발자의 개발 생산성을 높여준다.<br>
- 다만 Muli-Thread환경이기에 싱글톤 객체를 공유변수 체크를 하며 주의해서 사용해야한다.<br>
