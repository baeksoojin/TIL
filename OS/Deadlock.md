# Deadlocks

> 자원의 한계가 존재하기 때문에 process 할당을 기다리다가 계속 지속 되어 lock이 걸리는 상태를 deadlock이 걸렸다고 한다.


- Semaphore에서 wait operation이 지속되는 경우
- bridge crossing 처럼 brige에 양방향으로 데이터가 들어오면 lock이 걸리게 된다.
    - bridge를 비워줘야함(preempt : 한쪽에서 양보해서 다른쪽에서 bridge를 지나갈 수 있음, rollback : 양보를 해준 것으로 양보하는 라인의 데이터가 하나가 아니라면 그것들 모두 영향을 받음)
    - “양보”를 통해 해결하기에, starvation문제가 발생가능.

## Deadlock 특징 4가지

4가지가 동시에 만족한다면? Deadlock에 빠지게된다.

만약 하나라도 만족하지 않는다면? Deadlock이 걸리지 않는다.

1. Mutual exclusion(상호배제) : 어느 한 자원을 하나의 프로세스만 사용한다.
2. Hold and wait : 하나를 확보하는 상태에서 더 필요해서 기존의 것을 잡아두고 다른 것도 waiting하는 상태
3. No preemption = No Interrupt : 자발적으로 놔주기 전에는 interrup가 발생하지 않는다.  = 뺏어올 수 없다.
4. Circular wait : 서로 연결되어 기다리고 있는 상태

## Resource Allocation Graph

- request : a -> b 라면
    - a프로세스가 b를 요청하는 것.
    - 프로세스가 메인
- assignment : a →b 라면
    - a resource가 b 요청을 할당한 것.
    - 자원이 메인(Resource쪽에서 Process를 가리킴)

<p text-align="center">
    <img src="https://user-images.githubusercontent.com/74058047/236988105-ff9d2a77-1616-460f-8d1f-ee6c9dfd6de4.png" width="300" height="400">
</p>

- P1은 R1을 기다리고 있으며 R2를 할당받고 있는 상태
- P2는 R1을 할당받고 R2도 할당받았지만 R3는 기다리는 상태
- P3는 R3만 할당받음
- R4는 아무것에도 할당되지 않음.

<p text-align="center">
    <img src="https://user-images.githubusercontent.com/74058047/236987916-40a611e9-58c0-4f33-ae1d-784251388650.png" width="300" height="400">
</p>

- 상호배제 만족 : 1개의 자원은 1개의 프로세스만 접근가능
- Hold and Wait : 전부 hold & wait를 하는 중
- No preemption : 중간에 가로채지 못함
- Circular wait : circle 2개

<p text-align="center">
    <img src="https://user-images.githubusercontent.com/74058047/236987941-2f3b0ecb-b56c-4fa4-b737-23acb6ec60d2.png" width="300" height="400">
</p>

- Hold & Wait : P2와 P4는 hold and wait하지 않고 있어서 프로세스 이후 자원을 반납해서 다른 process가 사용할 수 있다.

⇒ Deadlock이 아님

---

## Basic Facts

- 사이클이 존재한다면 ? instance의 개수를 봐야함.
    
    1개의 resource라면 deadlock
    
    여러개라면? 가능성만 있다고 표현.
    

---

## Deadlock 핸들링

- never enter
    
    원천 봉쇄하여 아예 빠질 수 없게한다. 가장 강력!
    
    회피한다.
    
- Recover
    
    Deadlock이 될 수 있지만 빠져나오게한다.
    
- Ignore
    
    Deadlock을 신경끈다.
    

## Deadlock Prevention

- Mutual Exclusion →  nonsharable한 것에만 적용
- Hold 조건을 깨기 → 1. 원하는 process를 추가적으로 요청하지 못하게 추가 요청을 막는다., 2.  어떤 프로세스를 요청할 때는 갖고있는 자원이 없어야한다.
    
    자원에 대해 처음부터 확보하고 시작하기에 이용률이 떨어짐.
    
    **starvation**
    
    ex) 프린트는 가장 마지막이 대부분인데 처음부터 하나의 프로세스가 확보해놓고 마지막까지 놓지 않으니, starvation 가능
    

- No Preemption
    - 다 확보가 된 상태에서만 프로세스를 실행하도록 동작.
    - 뺐어오지 못한다? 내가 갖고 있던 자원을 반납해버려서 deadlock이 걸리지 못하게 한다.

- Circular Wait
    - 사이클을 깨줘야함
    - 번호를 부여해서 증가하는 순서로만 자원을 요청하도록해서 사이클을 막음.
    
    다만, 자원에 번호를 할당할때 조심스럽게(처리과정을 생각하며) 해야함
    
    ex) 프린트에 1번을 부여하면 안 됨.
    

## Deadlock Avoidance

사전에 최대 몇가를 필요로 할 것인지를 결정해놓고 **자원 할당 상태**를 게속 체크하면서 circular wait상태에 빠졌나 체크.

어느 프로세스가 여유분이 없다? 그러나 앞에 있던 프로세스가 자원을 반납하면? 처리가 가능함.

why? 순서가 정해져있기 때문에 성립이 가능함. → **Safe Sequence**

- safe → deadlock 빠짐
- unsafe → deadlock에 빠질 수도 있지만 아닐 수도 있음.

## Resource-Allocation Graph Algorithm

- Claim edge

실제로 필요한 단계는 아니지만 언젠가 필요로 할 것 같으니, 예약하는 것.

실제로 필요해 졌다면 request edge로 변경

- request edge

실제로 필요한 단계에 그리는 그래프 엣지

- Assgin edge

request 때문에 실제로 할당된 상태