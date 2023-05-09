## Process Synchronization

process의 동기화를 어떻게 처리할 수 있을지 알아보는 chapter이다. 동기화 기법을 이해하고 어떤 운영체제에서 어떠한 기법을 사용하고 있는지 알아봄으로써 Process Synchronization에 대해서 알아보자!

---

### Background

- Concurrent Access와 일관성 문제
    
    공유하고 있는 변수 혹은 데이터에 대해서 “동시에” process간 접근이 가능하다면 data inconsistency problem이 발생할 수 있다.
    
- Orderly execution으로 순서화
    
    동기화를 맞춰주기 위해서는 순서를 부여할 수 있다.
    
    - **count라는 변수(광역변수 - 공유)**를 사용해서 꽉 차있는 버퍼 공간의 개수를 담아놓는 역할이다.
    - 실행되면 consumer의 count변수는 1씩 감소되며 producer는 1씩 증가하는 로직이 될 것이다.
    
    **Producer**
    
    count 값이 buffer_size와 동일하다면 “full fill”상태로 do nothing이 while문의 조건에 의해서 적용된다.
    
    **Consumer**
    
    count 값이 0이라면 “null”이라는 것이기에 do nothing이 while문에 걸려서 처리된다.
    

---

### Race Condition

> 실행 과정 중에서 어떤 작업이 마지막에 실행됐느냐에 따라서 변수의 값이 변경되는 것을 의미한다.
> 

- count++

```nasm
register1 = count // ----(1)
register1 = register1 + 1 //----(2)
count = register1 // ----(3)
```

- count—

```nasm
register2 = count
register2 = register2 - 1
count = register2
```

high level language안에 작성된 Assembly language는 위와 같다.

이때 동시에 count=5인 상태에서 진행됐다면?

**race condition**이 적용된다.
<p text-align="center">
<img width="500" alt="image" src="https://user-images.githubusercontent.com/74058047/235358076-f3c9cb08-dbc7-4587-9cce-3bb16e8a1459.png">
</p>


해당 과정에서는 count는 count++ 연산이 먼저 실행돼서 6→5로 끝나는 결과가 나와야하지만 중간에 coutn—의 Critical Section의 code가 (1)번 과정보다 먼저 발생하게 되어 최종 결과가 5가 아니라 4가 되는 문제가 발생함을 알 수 있다.

### General Structure for Process Pi

- Critical Section, Entry Section, Exit Section

**Critical Section**에서는 주로 “공유변수”의 값에 접근 및 수정하는 로직이 된다.

그렇기 때문에 Critical Section의 작업이 동시에 발생하면 일관성 문제가 발생할 수 있다.

Critical Section에 진입하기 전에는 Entry section을 거치게 된다.

Critical Section이 끝난 후에는 Exit section을 거쳐 나오게 된다.

---

### solution

critical section에 하나의 프로세스만 접근하도록 하는 **3가지 조건이 존재한다.**

1. Mutual Exclustion : 상호배제

process Pi가 critical section에 들어갔다면 다른 프로세스는 접근이 불가능하다.

1. Progress 끊임없이 다음스텝 진행

critical section이 비어있을 때 “특정 프로세스를 선정해서 들어갈 수 있도록 만들어주는 작업이 연기 되어서는 안 된다”.

1. Bounded Waiting 한정된 만큼만 기다리기

n번에 언젠가 한번은 critical section에 들어갈 수 있도록 보장해야한다.

→ process 5개가 경합한다면 ? 모든 프로세스들은 5번중 1번은 들어갈 수 있도록 보장한다.

다만 프로세스의 실행속도는 고려하지 않는다.

### Peterson의 솔루션

- 동기화 솔루션 3가지를 만족한다.
- 특징
    - 2개의 프로세스 사이에서만 적용이 가능한 솔루션이다.
    - Load, Store instruction은 automic(원자성 - 쪼개질 수 없음) 해서 interruption이 들어오지 않는다.
- turn (Integer)로 차례를 나타낸다.
    - 이번에 어떤 process가 들어가는지
    - 각각 처음에는 상대방 process가 된다.
- flag(Boolean)으로 critical section에 들어갈 준비가 되어있는지를 나타낸다.
    - 의도대로 만들면 된다.
- do nothing을 while문의 조건으로 처리한다.
    - while문의 조건 : 상대방의 flag가 참이고 turn이 참일때.
        
        상대방이 준비되어있으며 차례도 상대방이니까 본 process는 donothing이다.
        

- Pi에 대한 코드

```nasm
do {

	//Entry Section
	flag[ i ] = TRUE;
	turn = j ;
	while ( flag[ j ] && turn == j) ;
	
	//CRITICAL SECTION
	
	// Exit section
	flag[ i ] = FALSE; 
	
	REMAINDER SECTION
} while (TRUE);
```

### Bakery Algorithm

- **번호표**를 부여해 순서를 제공한다.
    - 혹시라도, **번호표가 같을 때** process **id값으로 비교**하여 먼저 생성된 것을 먼저 실행시킨다.
- (ticket, processId) 쌍으로 번호표가 부여된다.
    - `(a,b) < (c,d)` ⇒ a와c를 비교하고 같다면 b와d를 비교하여 작은 것을 선택한다.
- **choosing** : 번호표를 받는 과정인지 아닌지를 나타내는 Boolean값
    - 자기자신이 번호표를 받는 과정에서는 번호표 비효를 하지 못하도록 (번호표가 할당받고 나서 비교가능) 할때 사용한다.
- number : Integer값으로 초기값은 0이다.

- process(i)가 critical에 들어가고싶을 때의 코드를 살펴보자
1. Entry Section

```nasm
do {
	choosing[ i ] = true;
	number[ i ] = max(number[0], number[1], …, number [n – 1])+1;
	choosing [ i ] = false;
	for ( j = 0; j < n; j++) {
	while (choosing[ j ]) ;
	while ((number[ j ] != 0) && (number[ j ], j) < (number[ i ], i)) ;
}
```

1. Critical Section → code작성
2. Exit section

```nasm
number[ i ] = 0;
//remainder section
```

종료시키기 위해서 실행이 끝난 프로세스의 번호표를 0으로 만들어준다.

### Synchronization Hardware 방식

- CPU가 하나일때
    
    **interrupt를 끄고** critical section에 들어간다면 문제가 해결될 것이다.
    
    하나의 critical section에 들어간다.
    
- 단점 : CPU가 여러개가 있을 때
    
    interrupt를 계속 on, off한다면 **too inefficient**하다는 특징이 있다.
    
    시스템의 확장성에 문제가 생길 수밖에 없다.
    

- atomic instruction = non interruptable
    - Test and Set
    - swap
    
    두가지 방법을 사용할 수 있다.
    

### TestAndSet

⇒ entry section에 활용한다.

- code

```nasm
boolean TestAndSet (boolean *target)
{
	boolean rv = *target;
	*target = TRUE;
	return rv:
}
```

target의 value는 false로 시작하는데 TestAndSet을 이용하면 test는 true가 되고 rv는 false가 된다. 따라서 return값은 (Lock value)는 참이되어 critical section에 들어가도록해준다.

- process

process1이 TsetAndSet을 시켰는데 바로 다음에 **process2가 TestAndSet이 실행된다면 rv가 True가 되고 target은 True가 되고 return True이다.**

- TestAndSet와 While문
    
    TestAndSet의 return값(Lock의 값)이 True일 때 while문의 조건이 참이 되어 → do nothing
    
    Busy waiting처리를 한다. **(Lock이 false(먼저들어간 프로세스는 Lock이 False)일때 풀린다)**
    
    while문을 나오면서 critical section이 실행된다.
    

- 전체코드
    
    ```nasm
    do {
    	while ( TestAndSet (&lock )) ; //entry section
    	/* do nothing
    	// critical section
    	lock = FALSE; //exit section
    	// remainder section
    } while ( TRUE);
    ```
    
    - mutual은 당연히 만족
        
        먼저 TestAndSet을 실행한 하나의 프로세스만 Lock이 false.
        
    - Progress조건
        
        Lock을 False로 만들어줘서 기다리던 애들중에 하나를 바로 실행.
        
    - **Bounded Waiting  조건 (X)**
        
        2개 이상의 프로세스 : if 4개였다면 lock이 False일 때 바뀌는데 기다리던 3개중에 랜덤으로 하나만 재수좋게 critical section에 들어간다. 따라서 보장은 시켜주지 않기에 Bounded 조건은 만족하지 않는다.
        
    
    ### Swap Instruction
    
    - Swap
    
    ```nasm
    void Swap (boolean *a, boolean *b)
    {
    	boolean temp = *a;
    	*a = *b;
    	*b = temp:
    }
    ```
    
    → 서로의 권한을 뒤바꾸는 코드
    
    - 공유변수 Lock → 초기값은 False
    - key → local 변수로 Pi안에서만 인정받는 변수이다.
    
    - 전체 code
    
    ```nasm
    do {
    	//entry section
    	key = TRUE;
    	while ( key == TRUE)
    		Swap (&lock, &key ) .. ;
    
    	// critical section
    
    	lock = FALSE; //exit section
    
    	// remainder section
    } while ( TRUE);
    ```
    
    key값이 True이고 lock이 false일때 while문에 들어가서 서로 값이 변경되면 key가 false가 되어 while문을 빠져나온다.
    
    lock의 초기값이 False이기에, while문이 False가 되어 맨처음에 실행한 process가 즉 가장 먼저 critical section에 들어간다.
    
    이후, lock이 False로 변경된다.
    
    - mutual은 당연히 만족
        
        먼저 Swap을 먼저 실행한 하나의 프로세스만 Lock이 false.
        
    - Progress조건
        
        Lock을 False로 만들어줘서 기다리던 애들중에 하나를 바로 실행.
        
    - **Bounded Waiting  조건 (X)**
        
        2개 이상의 프로세스 : if 4개였다면 lock이 False일 때 바뀌는데 기다리던 3개중에 랜덤(3개에 모두 기회를 준것이기에 순서를 보장하지 않음)으로 하나만 재수좋게 critical section에 들어간다. 따라서 보장은 시켜주지 않기에 Bounded 조건은 만족하지 않는다.

## semaphore

> 동기화를 위한 툴이며 busy waiting이 필요없는 특징이 있다.
> 
- busy waiting
    
    while → do nothing처리
    
- wait, signal
    1. wait operation → s≤0 → while (do nothing)으로 entry() 처리
    - 정의된 대로 의미상으로는 위와같고 실제로 구현될 때는 do nothing while문을 넣지 않아서 busy waiting이 적용되지 않는다.
    1. signal operation → s++ logic으로 exit() 처리
- signal과 wait을 사용하면 순서부여가 가능하다.
    
    signal CPU → interrupt를 막음.
    
    multiple CPU → spinlock( = wait and signal)을 사용.
    

### 고려사항

critical section의 작업이 오랫동안 발생한다면 ? → 차라리 busy waiting을 하기보다 context switching을 하는게 낫다고 볼 수도 있음.(waiting queue에서 기다리던 process를 가져오는 처리)

만약 critical section의 코드가 짧거나 가끔 발생하는 operation이라면 ? → busy waiting 방식이 더 좋을 수 있음.

## semaphore Implementation

- waiting : queue에 넣어 block처리 : Entry section
    - block이 된 프로세스들이 기다리는 큐
    - code
- signal : Exit Section
    - wakeup → queue에서 가져오는(remove) 처리 이후 동작
- value → 1로 시작해서 처음 동작때는 block되지 않음

### Deadlock and Starvation

- Deadlock
    - 2개의 프로세스가 동시에 Lock이 되어 아무것도 실행되지 못하는 상태를 Deadlock 상태라고한다.
    - Starvation문제가 발생할수있다.

---

## 동기화의 고전적인 문제들

### Bounded-Buffer Problem → starvation관련문제

empty : 초기값은 n으로 버퍼에 남은 값이 들어있음

mutex : 1로 시작 (≤0일때 block되지 않고 critical section에 들어감)

full : 초기값은 0으로 버퍼에 차있는 공간의 개수가 들어있음.

- consumer와 producer사이의 semaphore

![image](https://user-images.githubusercontent.com/74058047/236987706-3689e6ea-e7a5-472f-b6b8-826e8cc897fd.png)


버퍼의 개수가 3일때

pppp —-(1)

cccc ——(2)일때 “(1) → (2)” 가 진행된다면?

마지막 operation에서는 buffer공간 full=3 & empty=0이기에 wait(empty)에 걸려서 producer는 쓰지 못하지만 consumer가 signal(empty)를 처리하게 된다면 그때 producer의 p가 동작할수있다.

---

## Synchronization Examples

- Solaris → adaptive mutex
    - 실행중이던 porcess이며 critical section이 짧다면 busy waiting(spin)을 사용
    - cpu가 하나일때는 다른 작업은 sleep상태로(queue에 넣어서 기다리기)
    - lock을 사용해서 데이터를 읽고(여러개 가능) 쓰기(하나만 가능)를 제어
    - turnstiles(놀이공원 회전문 기법)를 사용해서 줄 세우기를 사용.

- Window Xp
    
    CPU가 하나 → interrupt를 사용
    
    CPU가 여러개 → busy waiting인 spin lock을 사용
    
    dispatcher → semaphore와 같이 동작하는 방법
    
- Linux
    
    critical section이 짧다 → interrupt
    
    single cpu → preemption or nonperrmption으로 intrerrupt를 제어
    
- Pthread
    
    Non-portable → read-write locks/ spin locks