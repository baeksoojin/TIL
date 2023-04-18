# CPU Scheduling

## Basic Concepts

기본 컨셉은 **“CPU의 이용률을 Maximum”** 으로 작업하도록하는 것이다.
<div align = "center">
<img width="130" alt="image" src="https://user-images.githubusercontent.com/74058047/231046390-9e771384-74b0-4004-b140-b255a76c6574.png">
</div>

CPU burst distribution

I/O burst(시간이 비교적 오래걸림)에서 **CPU burst(비교적 짧은 시간)** 를 기다려야한다면 기본 컨셉을 가져갈 수 없기에, multiprogramming으로 다른 process로 cpu 작업을 넘겨준다.

CPU는 길어봤자 2 millisecond를 차지한다.

<img width="300" alt="image" src="https://user-images.githubusercontent.com/74058047/231046490-976e424c-aff4-4e74-8c0a-70fac4bef240.png">

다 빨리 끝내고 놀고있으면 안 되니까 필요한것 ? <br>
**CPU Scheduling** 을 해줘야한다.

그렇다면 본격적으로 **CPU Scheduler** 에 대해서 알아보자!!

- CPU Scheduler
1. Switches from running to waiting state
2. interrupt : Switches from running to ready state [preemptive]
3. disk i/o 작업 끝 : Switches from running to ready state  [nonpreemptive]
4. Terminates

**Dispatcher**

 <img width="300" alt="image" src="https://user-images.githubusercontent.com/74058047/231046747-19c876bc-501d-4043-98e3-d68efcc3e039.png">

- context switching : context switching에 걸리는 시간에 short-term scheduler가 작동할 것이고 이를 dispath term이라고한다.
- cpu scheduler의 역할

   

---

## Scheduling Criteria

- Throughput : MAX일수록 좋음
- CPU utilization : MAX일수록 좋음
- Response time : reponse가 올 때까지 걸리는 시간이여서 Min일수록 좋음
- Waiting time : Min일수록 좋음
- Turnaround time : Min일수록 좋음

---

## Scheduling Algorithms

### FCFS

> first-com, first-served scheduling


먼저 온것을 먼저 처리한다.

- case1

    <img width="300" alt="image" src="https://user-images.githubusercontent.com/74058047/231046773-d07c4298-5616-4dc8-876f-f17671f241a9.png">

    CPU burst time과 순서가 다음과같을때 waiting time은? (0+24+27)/3 = 17

- case2

    <img width="409" alt="image" src="https://user-images.githubusercontent.com/74058047/231046789-af225985-2402-450e-a2f6-c94c687b29b1.png">

    CPU burst timewaiting time은? (6+0+3)/3 = 3

- 비교<br>
case1보다 case2가 AverageTime이 적다.

    **작은 작업을 먼저 처리하면 더 좋음을 알 수 있다.**

- Convey effect<br>
short process를 long process 뒤에서 기다리고 있는것.

### SJF

> Shortest-Job-First
> 

**average waiting time 측면에서 가장 적은 값을 가지는 최적의 방식**

- nonpreemptive방식
    
    ready queue만 고려하고 cpu의 남은 작업량은 고려하지 않음.
    
    Burst Time이 동일한데 동일 시점에 ready queue에 있다면 먼저 들어온것부터 처리한다.
    
    <img width="300" alt="image" src="https://user-images.githubusercontent.com/74058047/231046816-e2b4b5f0-2380-4621-bb8c-c72d28cd6837.png">
    
    (0+6+3+7)/4 = 4
    
    Context Switching : 3회
    
- **preemptive**
    
    cpu와 ready queue를 모두 고려하고 가장 짧은 남은 시간을 가진것부터 처리한다.
    
    > Shortest-**Remaining**-Time-First
    
    <img width="300" alt="image" src="https://user-images.githubusercontent.com/74058047/231046849-38fa8843-2502-4825-ac48-dc96cceb1d44.png">
    
    (9 + 1 + 0 +2)/4 = 3
    
    Context Switching : 5회 → overhead(유용한 작업 처리시간이 아닌시간)
    
    *nonpreemptive에 비해서 Average time은 줄었지만 **Context Switching** 이 늘어난 것도 고려해야한다.*
    

## Determining Length of Next CPU Burst

τn+1 = αtn + (1- α) τn 식을 활용해서 estimate<br>
오래될수록 반영하는 비율이 적다.<br>

<img width="300" alt="image" src="https://user-images.githubusercontent.com/74058047/231046887-20f3c1b1-8a60-41f7-89b1-fef1b5ac4afd.png">



## Priority Scheduling

> smallest integer를 가장 높은 우선순위로 하여 queue안에서 먼저 처리하는 cpu process 처리 스케줄링 방식이다,.
> 

- CPU 는 높은 우선순위를 가지는 것부터 처리를 진행한다.
    - Preemptive : cpu를 할당받아서 처리
    - Nonpremmptive : 한번 시작되면 preempted될 수 없음
- SJF : priority는 smallest job을 우선순위로 하여 처리한다.

**issue**

큰 job의 경우에는 적은 job을 가지는 것을 계속 먼저 처리하다보면, CPU를 할당받지 못하는 문제가 발생할 수 있다. ⇒ **Starvation problem**

어떻게 해결해야할까?  **Aging**을 solution으로 한다. → 오래되면 priority를 높여주면 된다.

## Round Robin

> 정해진 time quantum 만큼만 cpu를 사용할 수 없고 끝나지 않는다면 다시 뒤로 전송하는 방식
> 

- 들어온 순서대로 하지만, 시간이 정해져있음
- 전체가 n개의 process라면 → n/process 만큼의 시간을 할당받음 : q time
- 최대한 (n-1)q만큼 기다릴 수 있음 → n개라면 자기 빼고 나머지 n-1이고 하나당 실행 타임이 q(time quantum value)일때
- q large → 시간 분할이 없으니, FIFO와 동일
- q small → context switching이 필요 : overhead

**특징**

평균 SJF보다 turnaroud time이 높지만, response time이 더 좋다는 특징을 가진다.

**average waiting time** ? 각 프로세스마다 얼마나 기다렸는지 계산해서 합을 다 더해서 4로 나누면 평균을 구할 수 있음.

rule of **Thumb ⇒ 90% 정도가 되도록 q를 구하는 것이좋다.**

6, 3, 1, 7의 process time이라면 → 17/4 정도

## Multilevel Queue

- ready queue is partitioned
    
    foreground(interactive) → response time이 좋은 RR을 사용
    
    backgound → batch 작업으로 들어온 순서대로 FCFS 적용
    
- cpu를 두개의 프로세스에서 적절히 나눠서 처리해야함.
    - 치우쳐지면 starvation이 발생가능
    - Time slice 진행

### 우선순위

<div align = "center" >
<img width="300" alt="image" src="https://user-images.githubusercontent.com/74058047/232660128-a2e1c567-06f6-4351-b902-afa4996a60d5.png">
</div>

1. system process → 2. interactive process → interactive editing process → 4 batch process → 5. student process

cpu time을 분산시켜줘야하는데 각 queue의 작업을 처리하도록 정책을 정해서 적절히 나눠줘야함.

(만약 위의 과정이 끝나야 아래의 과정이 진행하도록 하면 starvation이 발생가능해서 항상 주의)

## Multilevel Feedback Queue

multi queue의 동작을 어떻게 설정할 것인가?

queue의 개수

demote, upgrade

io, interrupte 등의 작업이 끝났을 때 어디로 갈지 결정

<div align = "center">
<img width="300" alt="image" src="https://user-images.githubusercontent.com/74058047/232660153-86bdbe6f-65b5-4ebd-a091-dbde496d4b4a.png">
</div>

cpu를 할다받고 안끝나면 아래의 16 q를 가지는 queue에 줄서고 또 안 끝나면 FCFS로 들어가도록함(여기서는 무조건 끝날때까지 진행)

cpu가 processing power를 어떻게 해줄 것인가?가 고려될 수 있음

8인 q를 가지는 queue에게 다 할당하면 starvationa문제가 발생할 수 있음을고려해야함

- 위의 예에서 Shortest Job First Algorithm인가?
    
    RR이여서 먼저 들어간게 먼저처리되는 부분에서는 가능하지만 사실 정확히 보면 먼저 끝나면 먼저 나가니까 priority algorithm이라고도 할 수 있다.
    

## Multiple-Process**or**(cpus) Scheduling

- asymmetric : master, slave 사이에서 Master가 정해준것을 slave가 실행
- Homogeneous : 모두가 동등.
    - push : 작업량을 놀고있는 cpu에게 전달
    - pull : 작업하던게 끝나면 자기가 끌어와서 load balancing을 처리

**issue**

- **Cache hit ratio**
    - affinity : 특정 cpu의 입장에서 cache에 관련 process의 데이터가 많이 있을 때 해당 cpu에게 전달하는 방식을 고려했을 때 affinity가 높아진다.
    - 항상 담당했던 cpu에게 전달하면 hard affinity
    - guarantee를 보장하지 못하지만 최대한 하도록 노력한다면 soft affinity
    

## Real-time scheduling

- Hard real-time : realtime을 보장
- Soft real-time : 보장은 못하지만 pritoriy는 부여해서 노력

## Operation System Examples

- solaris 2 scheduling
    
    real-time이 중요한 것을 맨 위에 놓고 system service를 그 아래에서 작업하고 그 아래를 interactive한 것을 진행
    
- 우선순위 결정지표

<p align="center">
<img width="400" alt="image" src="https://user-images.githubusercontent.com/74058047/232660206-59cbfdb5-5b20-485e-b767-8aca54fc45c2.png">
</p>

- time quantum expired : q time이 지난 후 다시 실행되는 경우
- return from sleep → 끝났다가 다시 깨어나는 경우

**특징**

- 정책 : 반비례

우선순위가 높을수록 q time을 적게 주고 낮을 수록 높게 줘서 우선 순위 높은것만 처리하지 못하도록 할 수 있음.

⇒ interactive(우선순위가 높은것이 대부분)한 response time은 좋아지고 cpu의 throwput이 좋아짐

**왜 좋아질까? 에 대해서 생각해보기!**

## Linux Scheduling

**Time sharing**

- process마다 credit을 가지고 있는데 신뢰도가 클수록 cpu 할당을 가장 먼저 받음
- 먼저 다 신용값을 할당해줌(기반 : 과거실적 및 우선순위) : 초기화
    - credit은 할당받으며 점점 줄어들고 다시 높은 것을 결정해서 cpu할당
    - 다 끝나면 다시 초기화 진행

**Realtime**

soft real-time을 가짐 → FCFS와 RR
