## 운영체제란 무엇인가?

> A program that acts as an intermediary between a user of a computer and the computer hardware.
> 

사용자 혹은 사용자의 프로그램과 컴퓨터의 하드웨어 사이에 interface or intermediary라고한다.

즉, **사용자가 컴퓨터 하드웨어를 잘 사용할 수 있도록하는 interface가 운영체제**이다.

------

## **운영체제의 두가지 역할 : Definition**

resource allocator → 자원의 사용을 “공평”하게 “효율”적으로 사용하도록한다.

control program → program을 실행할 때의 error 를 **방지하고** 부적절한 프로그램을 **제어한다**.

“Everything a vendor ships”라고 할 수 있고 kernel(컴퓨터를 켰을 때 항상 돌고있는 프로그램)이라고 할 수 있는데, 이처럼 “No univerally accepted definition”의 특징을 갖는다.

- goal
    - Execute, solving → 사용자의 프로그램을 “실행”, 그리고 문제를 쉽게 “해결”
    - make the computer system convenient → 효율적으로 “편리”하게 사용하도록함
    - efficient → 효율적으로 관리

## computer system 의 구분
    
    
- hardware : 눈에보이는 컴퓨터 시스템을 구성하는 모든 것들
- operating system :  하드웨어를 사용하는데 있어서 여러 어플리케이션이나 사용자를 control, coordinate
- Application : 프로그램(system programs :  program을 만들고 실행시키고 프로그램을 수정하는데 필요한 것들을 부름. application programs : game or database system 등)
- User : 사람 혹은 기계 혹은 컴퓨터

- 세계 최초의 컴퓨터

    1만 8000개의 진공관을 이용해서 연산작업을 수행.

    공룡컴퓨터 에니악.

    탄도궤적 계산, 일기예보 등에 활용.

    카드에 일일이 입력하고 테입드라이브를 사용해서 모아놨다가 빛을 쏴서 인식되도록함.

- computer startup(power button 이 눌려질때 실행되는 것)
    
    bootstrap program : ROM(read only memory), EEPROM(수정하려면 할 수는 있지만 쉽지는 않음) → firmware
    
    initialize(하드웨어와 소프트웨어 사이 : 휘게 하려면 휠 수는 있지만 부드럽지는 않음)
    
    loading kernel and start execution(kernel의 동작) →loads
    
##  **Computer-System operation**

CPU(one or more), controllers, common bus를 통한 memory 공유사용

메모리는 주소값을 통해서 접근하는데 disk(데이터가 넘어옴)→memory→cpu로 흐름이 이어질 수 있다.


- I/O and CPU
    - cpu 작업과 입출력 작업이 동시에 일어날 수 있다.
        
    - ex) disk와 disk controller의 데이터 처리 과정 중에 cpu의 연산이 동시에 발생가능하다.
    - cpu가 memory의 데이터를 disk에 있는 buffer로 전송해주면 cpu는 다 끝난 것으로 간주. 실제입출력작업(buffer의 데이터를 disk로 옮기는 작업, disk의 데이터를 buffer로 옮기는 작업.)의 경우, Contoller의 역할이다.
        
        입출력작업이란 : disk와 controller작업까지의 범위에서 일어나는 것. cpu는 관여하지 않음.
        
    - DMA 테크닉을 활용해서 buffer에서 한번에 memory로 옮김 (buffer가 다 쌓이면 바이트단위가 아니라 덩어리로 cpu에게 interrupt로 알려줌)
    - 읽기(Input) : disk에서 데이터를 가져와 controller로 전달. 쓰기(Output) : controller buffer에서 disk로 조금씩 내보내는 작업
    - device마다 해당되는 controller가 있고 각각의 local buffer를 가지고 있음.
- DMA controller는 bus를 통해서 memory에 쓰게 됨.(byte단위가 아니라 덩어리로)
- “**interrupt**”를 발생시켜서 io작업이 끝남을 CPU에게 알려줌.
- CPU는 memory에서 읽어와서 연산을 수행.

**I/O은 CPU의 작업과 동시에 발생이 가능하고 만약 io작업이 끝났다면 interrupt를 사용해서 알려줘야 cpu가 local buffer의 데이터를 메모리에 저장하고 연산에 사용할 수 있게 된다.**

interrupt를 통해서 운영체제의 프로세스가 동작하는데 이를 interrup driven 방식이라고한다.
    

##  **interrupt driven**
    
interrupt를 여러종류(어떤 경우에는 harddisk쪽, 어떤 경우에는 monitor쪽)가 있을 수 있을 때, 각 interrupt마다 처리해주는 방법이 다른데 어디에 해당 처리 프로그램을 가지고 있냐(주소)에 대한 정보는 **interrupt vector**라고한다. (어떤 유형의 인터럽트인지 파악하는 단계)

interrupt가 걸린 시점을 save해서 처리 이후 다시 하던작업을 실행한다.(인터럽트 처리이전 기존의 처리과정 저장하는 단계)

interrupt를 처리하던 중에 interrupt가 새롭게 발생할 경우, “우선순위”를 비교하여 다 끝나지 않더라도 우선순위가 큰 것부터 처리한다.(실행하는 단계에 발생가능)

- trap : software적으로 발생한 에러를 처리할 수 있도록 신호를 줄때도 사용한다. 혹은 의도적으로 필요해서 system call 을 활용하기 위해서 발생시킴.
- 운영체제의 작업은 “interrupt”에 의해서 동작한다. = **interrupt** **Driven**

## interrupt handling
- interrupt가 발생하면 가장 먼저 하던 작업의 데이터들을 저장하고(각종 register값이나 program counter값을 저장)
- PC에 저장(Program Counter는 다음 주소값을 저장)
- interrupt Service Routine을 실행해주는데 어떤 인터럽트냐에따라서 다르게 처리$
- 실행이 끝나면 다시 PC의 값(돌아올곳의 주소)로 돌아가서 하던 작업을 마무리

    (interrupt cycle)

- 어떤 유형의 interrupt가 발생했는지 아는 방법 2가지
    1. **Polling** → *다 접근*하며 물어보는 방식
    2. vectored interrupt system → interrupt를 실행해주는 루틴이 몇번째에 존재하는지 interrupt를 가지고 살펴볼 수 있는 table과 같음. interrupt service routine : 각각의 처리할 프로세스

- interrupt timeline

    user process를 실행하는데 io작업이 끝나면 interrupt를 발생시킴.
    
- structure
    - sync : io가 시작되면 controller는 io가 끝날때까지 아무것도 못하고 기다리는 것.= cpu가 다른일을 하지못함. = 다른 ip작업은 동시에 진행할 수없음.
        
        어떻게 기다리냐? 
        
        wait instruction을 사용할 수 있음.(다음번까지 idle)
        
        wait loop을 사용가능.(메모리의 어떤 주소에 접근하는지만 계속 반복해서 알려주는 작업만 계속 진행함. 다른일은 못함)
        
    - async : cpu는 여러개의 io작업을 동시에 처리할 수 있도록함.
        
        비동기식 io보다 훨씬 효율적임
        
        대신 현재 작업중인것을 잠시 중지하고 다른것을 처리하는 것.
        
        system call
        
        device-status table를 둬서 각각의 io device들이 놀고있는지 바쁜지 혹은 줄서있는지 관리해주는 table.
            
    
- device-status table
        
    모든 io device의 상태를 관리하고 어떤 작업을 하고있는지를 표시하고 프로세스가 처리할수있도록 도움.
        
- DMA(Direct Memory Access)
    - byte단위로 발생시키면 너무 비효율적
    - 한꺼번에 고속으로 buffer가 차이면 메모리로 보내주는 방식이없을까? → DMA방식.(DMA방식 자체만 보면 CPU의 관여는 없음)
        - interrupt는 block단위로 한번에 발생.


## storage Hierarchy

- 계층적구조

위로갈수록 speed가 빠르고, 가격이 비싸다, Volatility(main memory까지는 전력이 꺼지면 사라지는 특성이 존재함), 전기적 vs 기계적 ( register, cache, main memory까지는 전기적이지만 disk까지 내려가면 harddisk를 동반할 수 밖에 없음) 

- caching 기능

아래에 있는 데이터를 그 윗단에 있는 storage로 옮기는 것을 캐싱기능이라고한다.

- Access-time(ns)

register(0.25-0.5) < cache(0.5-25) < main memory(80-250) < disk storage(5,000,000)

아래로 내려갈수록 접근시간이 차이가 나는데 disk로 가면 접근시간이 매우 커진다. 따라서 될 수 있으면 원했던 데이터는 main memory까지라인에서 줄 수 있으면 좋다.

## storage structure

- Main Memory - CPU가 주소를 가지고 직접 접근할 수 있는 저장 기기
- Secondary storage - main memory가 아닌 큰 nonvalatile메모리를 의미하고 보통 magnetic disk가 포함됨
- Magnetic disks - disk의 표면은 track으로 이루어져있고 track은 sector로 나눠지는데 읽기 쓰기 작업이 sector단위로 일어난다.

![image](https://user-images.githubusercontent.com/74058047/224628667-798380cd-93d7-4f8b-8712-f467089a37ea.png)

→ head는 platter가 돌 때 원하는 부분을 발견하면 해당 sector를 읽기, 쓰기를 하게되는데 그 간격이 매우 작아서 header와 platter가 닿을 수 있는 단점이 존재할 수도 있음. header가 매우 가볍기 때문에 빠른속도로 작업이 발생할 수 있음

- Optical disk → 표면을 태워서 1,0을 구분하는 방법. header가 비교적 무거워서 sector를 찾는 과정이 느릴 수 있지만, 읽고 쓰는 과정은 매우 빠름. magnetic disk에서는 head crash가 발생할 수 있지만, head와 platter의 간격이 비교적 커서 잘 발생하지 않는다는 장점이 존재함.

## Caching

> 빠른쪽의 저장기기로 미리 옮겨놓고 사용하는 것
> 

- Multiprocessor 환경에서의 cache coherecy
    
    다른 process들은 system bus를 모니터링하다가 system bus에서 오고가는 데이터들중 바뀐 데이터를 각자 바꾸면서 캐싱기능의 일관성 문제를 해결한다.
    
    여러개의 네트워크에서 작업을 할때는 더 어려워짐.
    

## Operating System Structure

- Multiprogramming
    - memory에 여러개의 작업이 존재할 수 있도록 해놓고 각각 처리할 수 있도록 CPU가 스케줄링해주면서 동작한다.
    - job1에 대한 i/o 작업을 하고 있을 때 job3에 대한 요청이 들어오면 그것을 처리해줌
- Timesharing(multitasking)
    - 시간을 나눠서 특정 시간을 할당해주면서 돌면서 처리해주는데 Response time < 1 second
    - 매우 빠르기 때문에 동시에 처리되는 것처럼 느껴짐
    - 1-100까지 메모리공간을 할당해주고 작업이 끝나면 2-200까지 차례대로 memory를 swapping 해주면서 작업을 진행
    - virtual memory기술을 사용해서 사용자의 입장에서 무한히 큰 공간을 사용하는 것처럼 가상의 메모리를 제공(물론 실제로 무한한게 아니라 swapping을 사용하겠지만)

## Operating system Operations

- 운영체제는 trap을 처리해줘야한다.
- inifite loop를 중지해주는 역할을 해줘야한다.
- Timer를 사용해서 무한루프를 방지한다.
    - interrupt를 작업 전에 세팅하고 OS는 counter가 작업을 처리하면서 감소하게 되는데 이때, counter가 zero가 되었는데 끝나지 않는다면 interrupt를 발생시키고 set up before로 만들도록한다.
    

## Transition form User to Kernel Mode

- Dual- mode(User, Kernel)
    
    사용자의 프로세스는 User mode에서 동작하고 운영체제의 프로세스는 Kernel Mode에서 동작하도록 dual-mode를 가지고 처리되어야한다.
    
    mode bit에 의해서 dual-mode 처리가 되는데 mode bit이 1일때는 User Process가 진행되고 0일때 Kernel Mode가 진행된다.
    
    ![image](https://user-images.githubusercontent.com/74058047/224628565-ee98b2fb-9773-4bb1-81ea-a5e7505a50df.png)

    

## Process Management

> 현재 실행중인 Program의 작업 단위를 Process라고한다.
> 

process관리에서 있어서도 OS가 동작한다.

- Process가 실행되기 위해서 자원을 요청하고 처리하고 반납하는 과정에서 관여한다.
- Single-threaded : 쓰레드가 하나일때 하나의 PC가 존재하고 하나씩 순차적으로 실행한다.
    - PC가 가리키는 다음 주소값을 가져와서 해당 작업을 하나씩 실행한다.
- Multi-threaded : 각 thread만큼 PC가 있고 작업이 일어난다.
    - 그렇다고 CPU가 동시에 여러개를 처리한다는 것이 아니라 하나씩 작업을 실행하는 것이지만, 각 thread가 active하게 실행하는 동안에 다른 것을 처리할 수 있음

## Process Management Activities

- synchronization
- communication
- deadlock (먹통상태) handling
- creating and deleting

## Memory Management

- all data in memory
- 실행 → all instruction in memory
- memory management → 메모리의 어느 파트를 누가 언제 어떻게 사용하는지 관리해야 효율적으로 사용이 가능.
    
    언제 메모리를 차지하고 있는 것을 아래의 계층으로 내려보낼 것인지 등을 관리해야함
    

## Storage Management

- uniform, logical view
    - speed, capacity, access method, transfer rate 등이 모두 다른 다양한 종류의 디바이스가 존재하지만, 일관적이고 논리적인 뷰를 제공해줘야한다.
    - file관리자

- File-System management
    - directories 형태로 관리
    - 접근제어 기능을 제공해야한다(누가 접근이 가능한지)
    - 생성 및 삭제를 해줘야하고 용량, 유형 등을 보여줄 수 있어야하며 파일들을 매핑(disk에서 main memory로 올려주는 기능 등)시켜주는 기능, backup 기능이 필요하다.

## Mass-Storage Management

- main memory안에 들어갈 수 없는 큰 사이즈의 데이터를 disk에 저장.
- 어떻게 운영하냐에 따라서 성능에 영향을 준다.

Entire speed of computer operation hinges on disk subsystem.

## I/O Subsystem

- Controller안에 Memory공간인 buffer가 존재함
- buffering, caching, spooling
    - buffering : 속도차이가 다른 경우 완충작용
    - caching : 속도가 더 빠른쪽으로 미리 보내놓는 작업
    - spooling : 어떤 작업을 출력값을 바로 다음 작업으로 이어지는 것
- general device-driver interface : 공통적으로 사용될 때의 그룹
- 각 기능별로 분리해서 접근 : driver로 구분

## Protection and Security

- Protection :운영체제에 의해서 정의된 모든 자원에 대한 보호를 하는 것(보호)
- Security : 외부적인 공격에 의해서 막아주는 역할을 하는 것(보안)

id값을 사용하거나(pk) 그것과 관련해서 모든 파일이나 프로세스에 대한 접근 제어를 해야한다.

group id를 형성하는 경우도 있다.

privilege escalation : 권한을 더 많이 주는 행위

## Computing Environment

office environment

- 정해진 시간만큼 한 사용자의 작업을 처리해주는 timesharing방식을 사용했었음
- portal로 발전

home network

- modem
- firewalled, networked

client-server

여러 client는 network를 통해 server에 접근

## Peer-to-Peer Computing

모든 컴퓨터는 서버가 될 수도있고 클라이언트가 될 수도 있어서 동등하다고 해서 peer to peer라고한다. 

**discovery protocol** : 원하는 기능을 어떤 것이 가지고 있는지 하나씩 물어보며 처리하는 방식이다.

## Web-Based Computing

**Load Balancer** : manage web traffic among similar servers

수강신청 등과 같은 작업을 웹이 처리할 때 트래픽이 몰릴 수 있으니 이때 부하 분산작업을 처리하기 위해서 load balancer를 사용

## Cloud Computing

cloud service

- network를 통해서 공유자원에 접근하고 필요한만큼 사용할 수 있음.
- SaaS > PaaS > IaaS(servere, storage 등)

provider

- Control, Manage




---

## 정리

- local buffer에 무언가 들어와서 cpu의 처리가 필요한 것을 CPU에게 알려주는 방법은? : interrupt

- buffer가 차이면 메모리로 한번에 보내는 방식은? : DMA

- word단위 → 데이터를 묶은 단위인
    
    block단위 → word를 여러개 묶으면 block단위
    
    DMA는 block단위로 전송. 다만, system마다 크기가 다르다.

- storage, process, thread 등등의 작업에 운영체제가 관여하여 Control, Manage를 수행.

