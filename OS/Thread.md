## Thread

- basic unit of CPU
    
    ThreadID, program counter, register set, stack space를 가짐
    
- peer thread끼리 서로 같은 자원을 공유하고 있음
- heavyweight process : 더이상 쪼개지지 않는 task or process로 one thread를 가지는 프로세스

**LAYER 측면에서 봤을 때**

- user level thread : user 관련 관리 스레드로(종료나 시작 등)을 system library에서 미리 제공 (a set of library안에 thread생성 기능이 이미 만들어져있음) : 커널에게 자원을 요청하거나 할 일이 없어서 쉽게 빠르게 부담없이 동작이 가능
- kernel level thread : CPU에게 던져지는 일감에 해당되기에 하나를 만들때마다 resource가 필요. 몇개이상 만들면 시스템의 부하가 발생가능. 최적의 thread의 개수를 결정짓는 것이 필요함.
- Hybrid : kernel level thread와 user level thread를 두개 사용. 서로의 개수가 동일하고 CPU는 kernel level thread만 신경쓰게 됨.

---

## Multiple Thread

**장점**

서로 datam pc를 공유하고 사용하기에 context switcing과정을 통해서 새로운 메모리 영역을 할당하는 과정이 매번 필요하지 않다.

Not Independent of one anohter하기 때문에 , 하나의 thread가 block되더라도 다른 thread가 처리해주는 장점이 존재한다.

---

## Multithreading Models

- Many-to-One

하나의 Kernel level thread가 존재. user thread가 경쟁해서 잡아야함.

- One-to-One

일대일로 mapping되어, 경합하지 않아도 된다.

다만, kernel level thread는 계속 만들 수 없음(자원적인 부분)

- Many-to-Many

kernel thread를 최적 개수로 한정시키고 그것을 user thread끼리 공유해서 사용

조금 더 나아졌지만, 계속해서 경합이 붙는 것은 마찬가지.

그렇다면, 중요한 일처리를 계속해서 하지 못할 수 있게된다.

- Two-level Model

중요한 부분은 하나의 kernel로 할당해서 처리.