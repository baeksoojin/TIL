# Process

**Process in Memory**

<img width="100" alt="image"  src="https://user-images.githubusercontent.com/74058047/228117675-c0f6ab99-2ce9-451d-8a47-39f1838039fe.png">

- stack에 temporary 데이터(local var)를 저장. 이때 메모리가 부족하다면, heap사이에 그 공간을 늘린다. → 화살표 영역으로 확장가능
- heap : dynamically allocated
- data : global 변수저장
- text : codes

## ****CPU Switch From Process to Process****

**State**

<img width="300" alt="image" src="https://user-images.githubusercontent.com/74058047/228117717-b6e587b3-eca6-41ef-8e2e-dffe1c0665b7.png">

- new : 생성된 상태 (누군가의 더블클릭에 의한 어플실행)
- ready : 실행 준비중인 상태(모든 자원이 확보되면 큐에서 대기중인상태)
- running : 프로세스의 실행중인상태 (큐안에서 차례대로 돌다가 순서가 와서 cpu를 할당받은 상태)
- waiting : 이벤트 발생에 대해서 대기중인 상태
- terminated : 끝난 상태

**PCB(Process Control Block)**

> 하나의 프로세스마다 할당되는 프로세스를 표현하는 블럭

<img width="100" alt="image" src="https://user-images.githubusercontent.com/74058047/228117738-518cd0f4-ac9e-419b-8134-e74b7ef4fb24.png">

- process state(ready, new,,,)
- program counter
- CPU register : cpu가 사용하는 레지스터
- queue에서 사용하는 포인터정보
- 메모리관리 정보 : base,limit register관련 정보, page,seg table관련 정보
- Accounting information(job, process no, CPU time used,limit)
- I/O status information

**Context Switch**

<img width="300" alt="image" src="https://user-images.githubusercontent.com/74058047/228117767-fc1939e4-848e-4592-b46f-fae030f8ef73.png">

> CPU의 프로세스를 다른 프로세스로 변경할때, old process의 PCB값을 저장하고 new process를 load하는 과정


ex)위의 그림에 대한 설명

PCB0 → PCB1 사이에 CPU의 스케줄링이 일어나고 다음에 실행시킬 것을 탐새갛고 발견했다면 PCB1에 대한 데이터를 모두 해당되는 레지스터 위에 복구를 해주는 과정이 Context-Switching이라고 할 수 있다.

- P0 : running → waiting → ready → running
- P1 : waiting(idle)  → ready(필요한 데이터 레지스터에 복구 : cpu만 확보못한상태) → running(cpu확보) → waiting

**Scheduling Queue**

<img width="300" alt="image" src="https://user-images.githubusercontent.com/74058047/228117798-370ff226-0218-494b-b32d-42db27dc8ea5.png">

- job queue : 모든 프로세스
- ready queue : ready상태인 프로세스들이 대기중인 공간
- waiting queue : cpu처리도중 다른 프로세스를 처리하기 위해서 대기하는 공간 → 중간에 I/O 작업 혹은 time의 expired, fork a child 등등의 경우가 들어오면 대기상태로 넘어가고 이후 ready로 다시 보내진다.

## Scheduler

<img width="300" alt="image" src="https://user-images.githubusercontent.com/74058047/228117819-52e3fcb6-4ad1-47c3-a35a-e41a8af412bd.png">

- Long-term scheduler : 누가 ready-queue에 올라갈 것인지 선택
    - multi programming : 메모리위에 올라와있는 작업의 개수인 degree를 결정짓는다.
- Short-term scheduler : 자원확보가 CPU만 안 된 ready queue에서 어떤 것을 CPU에게 할당할지 선택
    - CPU는 매우 빨라서 스케줄링 속도가 빠름

CPU 할당이후의 Process를 처리하는 스케줄러

- medium-term scheduler :
    
    → CPU에서 처리하다가 ready queue에만 계속 보내면 메모리 공간이 너무 많이 찰 수 있음
    
    → Swap out : (ready queue가 아닌 disk에 보내는 것)
    
    → Swap int :  memory로 swap하는 과정이 이후, 프로세스 사용시에 다시 필요하함. 해당 역할도 진행
    

- I/O process는 cpu연산은 많지 않고 input, output작업 위주로 실행
- cpu-bound process : very long CPU burst(지금은 2개)로 진행되는데 cpu 연산작업이 많음

**Process Creation**

- parent and child
    1. Share all : parent와 child는 모든 자원(memory: 실행 code와 data를 저장중)을 공유
    2. Share subset : 일부분만 공유하는 방식으로 큰 작업을 child만들어서 특정 부분을 실행하도록함
    3. Share no : 공유를 아예 하지 않아서 별도의 메모리 공간을 child에게 주는 경우가 발생함

- address space(memory 공간)
    - **fork()** : share all로 같은 프로그램과 같은 데이터를 공유하는 환경 → fork()
    - **exec()** : fork를 호출한뒤 별도의 자원을 할당해주기 위해서 → exec()를 바로 실행(별도의 메모리공간에 별도의 프로그램을 작업)
    
- ex)

```c
int main() {
	Pid_t pid;
	/* fork another process */
	pid = fork(); /* fork() 라는 System Call을 이용하여 pid 생성 */ 
	if (pid < 0) { /* error occurred */
		fprintf(stderr, "Fork Failed");
		exit(-1); }
	else if (pid == 0) { /* child process */
		execlp("/bin/ls", "ls", NULL);
	}
	else { /* parent process */
		/ fork() 다음에 ‘execlp()’ = ‘exec()’ /
		/* parent will wait for the child to complete */ wait (NULL);
		printf ("Child Complete");
		exit(0);
} }
```

**Process Termination**

- exit : 모든 결과를 parent에게 제공하고 자원을 운영체제에게 반납하고 최종적으로 종료(정상종료)
- abort : 강제로 종료된 경우로 child의 자원이 할당받은 것보다 넘친경우나 필요없어지거나 parent가 종료된 경우(cascading termination) 등이 존재함(비정상종료)

## cooperating process

- independent : 독립적인 프로세스
- cooperating : 서로 영향을 줄 수 있고 받을 수 있음(parent-child 대부분이 여기에 속함) → 서로 해끼치는 것은 하지 않는다는 가정하에서 진행
    - 장점
    1. 정보를 공유(메모리 공간을 같이 공유)
    2. 쉽고 빠르게 처리(분할정복 : 다만 cpu는 여러개여야함. multi CPUs)
    3. Modularity : 담당업무를 나눠서 처리
    4. Convenience : parallel로 처리돼서(parent-child)

**Producer-Consumer Problem**

- Producer(만들기) → (buffer : 쌓아놓을 곳) → Consumer(가져다가 쓰기)
    
    producer가 작업하지 않을때
    
    consumer가 가져다가 쓰지 않아서 buffer가 다 차서 producer가 일을 하지 못할때
    
- 예를 들어서 shared-memory solution인 경우를 살펴보면 아래와 같다.
    
    
    <img width="150" alt="image" src="https://user-images.githubusercontent.com/74058047/228117921-060b151f-2aab-430c-8ccb-6dbc5ba4effb.png">
    
    - buffer_size → 10 : 다만 n-1일때는 더이상 공간이 부족해 아무것도 진행하지 않고 while문만 돌게하기 위해 사용되어서 실제 사용할 수 있는 사이즈는 9개
    - Insert()
        
        ```c
        while (((in + 1) % BUFFER SIZE) == out)
        ```
        
        이동한 곳이 out이 된다면 buffer에 공간이 존재하지 않는 것으로(producer가 만들어놓는 작업을 진행했으나, consumer가 사용하지 않은 경우) 무한루프에 빠지게된다.
        
        ```c
        in = (in + 1) % BUFFER SIZE;
        ```
        
    - remove()
        
        ```c
        while (in == out) // do nothing - nothing to consume
        ```
        
        만들어놓은 것이 없으니 remove를 할 수가 없는 경우여서 무한루프를 돌게된다.