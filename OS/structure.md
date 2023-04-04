# Operating System Structure

## Operating System Services

> helpful to the **User**
> 
- UI - User Interface
    
    Batch(큰 그룹을 나눠서 처리. not interactive, Scheduling), CLI(Command-Line), GUI(Graphics User Interface)
    
- Program execution - 프로그램 실행.
    
    메모리 Load → Run → End
    
- I/O Operation
    
    다양한 종류의 i/o 작업을 지원
    
- File System  Service
    
    read, write, create, delete, search, list, permission
    
- Communication - process , network
    
    Shared Memory : 메모리를 공유하는 과정에서 읽기,쓰기 작업이 발생
    
    Message Passing : Message를 save, receive
    
- Error detection - OS는 error에 대한 action(에러처리 프로그램)을 실행
    
    debugging : 수정하는 툴

-----
    

**시스템의 효율적인 사용**

- 자원할당 : cpu, memory, data, i/o device 등등에 대한 할당 및 회수는 운영체제에서 제공한다.
    - 프로세스는 자원할당을 가장 먼저하며 프로세스 실행 및 종료 이후 자원을 반납하게 된다.
- Accounting : 컴퓨팅 자원을 누가 얼만큼 사용했는지를 기록
- Protection and Security : 컴퓨터 자원에 대한 접근제어

-----

## User Interface

### CLI

> Command Line Interface - 명령어 창에 직접 입력해서 작업을 진행하는 인터페이스 방식
> 

kernel, built-in 방식

- 명령어가 모두 컴파일된 상태에서 그것을 사용해서 실행(command built-in)시키는 방식.
    
    ⇒ 수정하거나 삭제할때 전체에 대한 명령어에 대해서 다시 모두 다 컴파일해야하기에 효율성이 떨어짐.
    
- system program 형태로 명령어들이 존재하기에, 해당 명령어를 실행시키기만 하면 됨.
    
    ⇒ 따라서, 명령어를 실행하거나 삭제하기가 쉬움.
    

### GUI

> User-friendly desktop metaphor interface


- window는 GUI가 기본이고 CLI가 옵션.
- Linux, Unix는 CLI가 기본이고 GUI가 옵션.

-----

## System Calls

> Programming interface to the services
> 

대부분 직접 사용하지않고 API를 호출해서 쓰고싶을때 형식에 맞춰서 사용.

대표적으로 Win32 API, POSIX API, JVM(Java Virtual Machine)에서 제공하는 JAVA API 등이존재.

ex) copy

A → B
![image](https://user-images.githubusercontent.com/74058047/224882844-789febfe-1c52-4408-927e-7c3c0e78ff00.png)


중간과정에서 수많은 System Call이 발생

ex) Standard API

![image](https://user-images.githubusercontent.com/74058047/224882859-dbdc0cea-d727-420a-9c0c-bb12798bab09.png)

- ReadFile() function in the Win32 API

file(target), buffer, bytesToRead(how many), bytesRead(지금까지 받아온 양), overlapping(혹시라도 i, o가 같이 진행되고있는지 체크)

- Implementation

number로 system call의 흐름을 제어

![image](https://user-images.githubusercontent.com/74058047/224883146-0b8fe3aa-e4c0-42ba-b57f-4c35b8f6394d.png)


어떤 API 형식에 맞춰서 어떤 특별한 function이나 instruction을 호출할 수 있는지 그리고 그 결과로 어떤 결과가 넘어오는지 체크

----


**Parameter Passing**

- Register

레지스터를 사용

- 전달되어야하는 parameter를 stack으로 처리

parameter를 push → 읽을때 pop

⇒ 두 경우는 사이즈를 미리 고정.

- Memory

일부분을 block 으로 잡아서 사용하면 일부 유동적으로 사이즈를 변화시킬 수 있음.

---

**Types of System Call**

- Process control
    
    종료, 실행, 생성, 삭제, (get, set) attributes, wait for time, wait event, signal event, 메모리 할당 및 회수
    
- File management
    
    create file, delete file, open, close, read, write, get file attribute, set 
    
- Device management
    
    reqeust, release(반환), read, write, reposition, get attribute, set, logically attach or detach device
    
- Information(시간, 버전 등의 정보) maintenance
    
    시간,날짜 get or set , system data get or set
    
    process , file of device attribute get or set
    
- Communication
    
    create, delete communication connection

---

## system programs

> some of them are simply user interfaace to system call
> 

- status imformation

registry : 사태정보를 저장하기 위한 레지스터들의 묶음

status : date, time, amount of available
memory, disk space, number of users

- file modification
    - text editors : text editor, to search contents of files
    - programming-language : compiler(한번에 기계어로 번역했을 경우를 컴파일러라고함 - high level language), assemblers( low level language를 기계어로 번역), interpreters(한줄한줄 번역)
    - loading : memory로 실행시키기 위해서 가지고 올라오는 것.
        
        절대 로더 : 한번 메모리로 가지고 오면 메모리 위치가 변화하지 않는다.
        
        relocatable loader : memory 위치가 변화할 수 있음
        
        overlay-loader : memory위로 올려주는 로더이지만 중첩기능이 포함되어있는 메모리 로더. 프로세스 하나가 사용중인 메모리를 다른 프로세스가 특정 빈 부분을 비었을 때 사용이 가능하다.
        
    - communication : 한 사용자가 다른 사용자에게 메시지나 이메일을 보내는 것. 원격 로그인 등

--- 

## system design

운영체제의 디자인 및 구조.

초기에 운영체제가 어떻게 나왔는지에 대한 내용. 사실상 내부구조는 운영체제에 따라서 다르지만 설계하려는 목적에 대해서 알아봄.

- 사용자의 목표와 시스템의 목표를 분리해서 생각
    - user goal : convenient to use, easy to learn, reliable, safe, and fast
    - system goal : 관리하기 쉬워야하고 디자인하기 쉬워야하고 신뢰성과 효율성을 지켜야한다. 에러가 없어야한다.
- policy와 mechanism을 확실하게 구분해서 설계
    - what : 무엇을 (이후에 변경이 가능)
    - mechanism : 어떻게
    
    policy는 바뀔 수 있는데 그것에 맞는 메거니즘만 준비해놓으면 이후 변경이 되었을 때 대처할 수 없음. 따라서, policy, mechanism을 분리해서 설계해서 모든 가능성을 열어둬야한다.
    

---

## system structure

- MS-DOS : disk operating system

main memory의 크기가 커봤자 614k였어서 제약사항이 많았음.

![image](https://user-images.githubusercontent.com/74058047/226505702-c5e2f930-7c39-4aa1-a146-5044758ace3b.png)

ROM BIOS(basic input output system)가 가장 마지막

**특징**

application program에서 단계를 다 거쳐서 ROM BIOS를 접근하다보면 memory가 가득 차서 문제점이 발생할 수 있다.

그래서, 바로 이동하게끔 하였다.

하지만, 잘 분리되어있다고 말할 수 없는 모델이다.

- layerd approach

![image](https://user-images.githubusercontent.com/74058047/226505743-625d0aab-aac1-4ff1-90a9-7d0a81f19d46.png)

layer별로 나눠서 구분.

위로 올라갈수록 기능이 커짐.

참고로) filer manager가 n-1의 layer에 있어서 UI에서 오는 system call을 동작.

**특징**

layer i에서는 0~i-1의 layer의 기능을 사용할 수 있지만, 역으로는 불가능하다.(아래로는 접근이 가능하지만 위로는 불가능)

따라서, 기능배정이 중요해지고 만약 기능배정을 잘못해서 file manager를 하단에 배치하면 device driver를 작동할 수 없기에 배치를 중요하게 고려해야한다.

- UNIX의 등장

![image](https://user-images.githubusercontent.com/74058047/226505805-027b5cf6-9f07-4114-a801-5eb19831e572.png)

**특징**

Kernel : 물리적인 계층의 위에 존재하고 system call interface 아래에 존재하게 된다.

kernel이 굉장히 많은 특징을 가지고 있다. 하나를 변경하기 위해서는 연관된 다른것들을 다 수정해야한다는 단점이존재한다.

따라서, system program과 kernel 두가지로만 분리되어있어서 발생가능한 단점을 극복한게 Microkernel System Structure

- Microkernel

![image](https://user-images.githubusercontent.com/74058047/226505812-cca2c2be-9259-4e71-b6c3-186f675c7f16.png)

**특징**

core : 경량화 되어있다.

⇒ 핵심적인 기능은 밖으로 (kernel밖)으로 빼졌다.

⇒ kernel과 외부의 모듈이 communication을 message passing으로 이루어지게 된다.

object-oriented approach

separate

message passing을 통한 커뮤니케이션(각 모듈끼리)

필요할때마다 메모리로 로딩시키고 다 사용하면 내려보내준다.

**장점**

extend가 쉬움, port가 쉬움, reliable(커널에서 돌고있는게 적다보면 그 안에 들어갈 에러가 줄어들어서 신뢰도가 커짐), secure(커널에서 돌고있는게 적다보면 그 안에 들어갈 에러가 줄어들어서 안전성이 커짐)

**단점** 

message passing과정에서의 overhead가 발생할 수 있음

**비교**

| microkernel | layerd |
| --- | --- |
| 모듈화 | 모듈화 |
| 더 유연함 | 밑에있는 레이어만 접근이 가능해서 밑의 level과 엮여있음 |

### virtual Machine

Virtural Machine

본격적으로 클라우드 컴퓨팅을 사용하기 시작하면서 각광받기 시작함.

![image](https://user-images.githubusercontent.com/74058047/226505829-a56d93f4-88d2-409d-95f7-7d3c7ad59b4c.png)

사용률이 10%밖에 안 되는데 모두 사용하면서 낭비할 필요가 없으니, 쪼개서 사용하자는 idea에서 등장

- 쪼개서 서로 다른 VM을 사용하고 서로 다른 OS 사용이 가능
- 가상화 툴은 하드웨어와 운영체제를 VM입장에서 봤을때 hardware로만 인식하게끔하고 각각의 VM은 별도의 운영체제를 사용하도록한다.

![image](https://user-images.githubusercontent.com/74058047/226505841-b54238fb-0612-4207-96ab-1d6b9706f881.png)

**최적화**

Before (베어메탈): 프로그램을 작동시키기 위해서 A를 최적화 시키면 B,C에 간섭이 일어나고 B,C는 최적화를 하지 못할 수 있었음

After (VM): 각각의 가상머신 안에서 최적화를 진행해줘서 서로의 간섭현상이 일어나지 않는다.

**하드웨어**

다만 하드웨어는 물리적인 영역으로 하나만 존재한다.

하지만, 가상머신들은 실제로는 그렇지 않지만 각자의 피지컬 영역을 가지고 있다고 생각함 : illusion

**How? 어떻게 분리된것처럼 느낄 수 있게하나**

1. CPU scheduling : cpu의 power를 (cpu 할당을) 나눠서 특정 VM에서만 사용하도록하여 CPU power를 분리한다.
2. spooling(입출력에 해당하는 작업이 동시에 일어나도록 하는 기술) and filesystem : disk를 분리해서 (ex, c drive, d drive 분리하듯이) 각각에 할당
3. operator’s console : 일반 사용자의 모니터를 이용. terminal server

**protection**

서로 독립적이여서 간섭하지 않고 영향을 받지 않는다.

research and development 환경 등을 분리

**단점**

구현이 어려움

### JVM : java virtual machine

![image](https://user-images.githubusercontent.com/74058047/226505847-d0ac878b-3957-4030-b856-9112aaa16e65.png)

- class loader는 외부, 내부 라이브러리 및 클래스를 사용
- interpreter를 사용해서 한줄씩 번역. → 따라서 약간 느리다는 단점이 존재했지만 속도 향상 기능들이 나오기 시작

### operating system  generation

- SYSGEN : 하드웨어에 마우스가있는지 용량이 얼마나 있는지 등등의 필요한 정보를 모두 수집하는 역할
- 정보를 ROM을 동작시키며 실행 : Bootstrap Program

이후, 부팅이 필요한데 Bootstrap Program을 사용.

ROM 안에 이미 다 저장이 되어있는데 cpu옆에 꽂혀있고 power가 들어오면 가장 먼저 ROM에 있는 것을 모두 실행 = Bootstrap Program이 동작함.


---


## InterProcess Communication (IPC)

> 프로세스가 소통하고 동기화하는 방식
> 

- Message Passing : Send(설계시 고정할것인지 가변하게 할 것인지 고정), Receive
    - communication을 위한 link가 있어야함.
- communication link
    - pysical(shared memory : 서로 공유하는 메모리를 읽고, 쓰고, bus, switch : cross bar switching 대표적, )
    - logical(direct[무언가를 거치지 않고 바로 send, receive] or indirect[무언가를 통해서 send, receive], syn or async, auto(무한대) or explicit(한정적)buffer)

- 고려사항
1. link를 어떻게 설계해 둘 것인지
2. 몇개를 연결시킬 것인지
3. 얼마나 많은 link를 every pair 사이에 둘 것인지
4. capacity of a link(bandwidth)
5. message의 size를 고정시킬 것인지 아닌지
6. 단방향 혹은 쌍방향

## communication models

- Message Passing vs Shared Memory

Message Passing : kernel이 우체국처럼 동작하고 한쪽에서 쓰고(우체통에 넣기) 한쪽에서는 편지를 가져오는역할(빼기)

Shared Memory : 사시에 공유된 영역을 두고 읽고 쓰는 방법

---

## Direct Communication

직접 연결되는 경우

다른것은 끼면 안 되고 바로 한 쌍끼리 서로 연결이 되어야한다.

- 한 쌍 안에는 오직 하나의 path만 존재해야한다. (서로 얽혀있으면 안 되기 때문)
- unidirectional 가능, 그러나 주로 bi-directional하다.

## Indirect Communication

mailbox를 사이에 두고 간접적으로 연결된 경우

communication을 원하는 peer끼리 같은 mailbox를 공유하고 있어야한다.

- mailbox를 통해서 가는데, unique한 id를 가져서 서로 중복되지 않는다.
- p1, p2는 서로 공유하는 메일 박스만 존재하면 여러개의 Mailbox를 가질 수 있다.

**Operation**

- mailbox를 활용하는 과정

mailbox를 create → send or receive → destroy mailbox

**Primitives**

- sned(A,message), receive(A, message)

**Mailbox sharing and Solution**

P1,P2 and P3 share mailbox A일때

여러개가 붙어있어도 receive를 실행시킬 것은 하나이다.

who gets the message ? → 하나를 선택해서 전달하고 누구한테 전달했는지 알려줘야함.

---

## Synchronization

### Blocking

**Blocking Send**

Direct 성질. Send하고 Receive할때까지 Block된다.

- 받는쪽이 받아갈때까지 보내는 operation을 계속 진행하며 blocking

**Blocking Receive**

Receive하려고 계속 Block 처리중인데 send하지 않아서 지속된다.

- 보내는 쪽이 보낼때까지 operation을 계속 진행하며 blocking

### Non-Blocking

- asynchronous
- 

다른쪽의 action을 기다리지 않음. 

그러기 위해서는 당연히 buffer가 필요함.

## Buffering

- Zero capacity : 서로 직접 연결이 되어 있어야 주고 받을 수 있는 경우로 blocking, synchronous(서로 동시에 연결되어있어야 서로 주고 받고가 일어남)
- Bounded capacity : finite length of n message
- Unbounded capacity : 절대 꽉찰일이 없어서 보내고 싶을때 보내고 받고 싶을때 받는경우로 non-blocking, asynchronous

---

## Client-Server Communication

- Socket
- Remote Procedure Calls = Remote Method Invocation(in Java)

## Socket

> 프로세스 간의 communication을 위한 최종단말로 endpoint
> 

- socket의 구성

IP:PORT → sort

서버끼리의 통신은 ip address와 port를 이용해서 원격으로 communication을 시작.

- 몇개정도는 고정

telnet : 23 port, ftp : 21 port

## Remote Procedure Calls

원격으로 다른쪽의 computer의 프로시저를 호출해서 결과를 넘겨받는 과정

> Remote Procedure call : RPC
> 

- client-side stub : server에게 RPC의 address를 요청하고, 하고싶은 것을 진행한다음에 client에게 최종 결과 전송
- server-side stub : 요청받은 RPC의 데이터를 전송해주고 output을 전송