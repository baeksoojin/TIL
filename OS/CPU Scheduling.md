# CPU Scheduling

## Basic Concepts

기본 컨셉은 **“CPU의 이용률을 Maximum”** 으로 작업하도록하는 것이다.

<img width="130" alt="image" src="https://user-images.githubusercontent.com/74058047/231046390-9e771384-74b0-4004-b140-b255a76c6574.png">

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




