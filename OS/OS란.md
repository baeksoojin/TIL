## 운영체제란 무엇인가?

> A program that acts as an intermediary between a user of a computer and the computer hardware.
> 

사용자 혹은 사용자의 프로그램과 컴퓨터의 하드웨어 사이에 interface or intermediary라고한다.

즉, **사용자가 컴퓨터 하드웨어를 잘 사용할 수 있도록하는 interface가 운영체제**이다.

**운영체제의 두가지 역할 : Definition**

resource allocator → 자원의 사용을 “공평”하게 “효율”적으로 사용하도록한다.

control program → program을 실행할 때의 error 를 **방지하고** 부적절한 프로그램을 **제어한다**.

“Everything a vendor ships”라고 할 수 있고 kernel(컴퓨터를 켰을 때 항상 돌고있는 프로그램)이라고 할 수 있는데, 이처럼 “No univerally accepted definition”의 특징을 갖는다.

- goal
    - Execute, solving → 사용자의 프로그램을 “실행”, 그리고 문제를 쉽게 “해결”
    - make the computer system convenient → 효율적으로 “편리”하게 사용하도록함
    - efficient → 효율적으로 관리

- computer system 의 구분
    
    
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
    
- **Computer-System operation**
    
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
    

- **interrupt driven**
    
    interrupt를 여러종류(어떤 경우에는 harddisk쪽, 어떤 경우에는 monitor쪽)가 있을 수 있을 때, 각 interrupt마다 처리해주는 방법이 다른데 어디에 해당 처리 프로그램을 가지고 있냐(주소)에 대한 정보는 **interrupt vector**라고한다. (어떤 유형의 인터럽트인지 파악하는 단계)
    
    interrupt가 걸린 시점을 save해서 처리 이후 다시 하던작업을 실행한다.(인터럽트 처리이전 기존의 처리과정 저장하는 단계)
    
    interrupt를 처리하던 중에 interrupt가 새롭게 발생할 경우, “우선순위”를 비교하여 다 끝나지 않더라도 우선순위가 큰 것부터 처리한다.(실행하는 단계에 발생가능)
    
    - trap : software적으로 발생한 에러를 처리할 수 있도록 신호를 줄때도 사용한다. 혹은 의도적으로 필요해서 system call 을 활용하기 위해서 발생시킴.
    - 운영체제의 작업은 “interrupt”에 의해서 동작한다. = **interrupt** **Driven**

- interrupt handling
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

---

## 정리

- local buffer에 무언가 들어와서 cpu의 처리가 필요한 것을 CPU에게 알려주는 방법은? : interrupt

- buffer가 차이면 메모리로 한번에 보내는 방식은? : DMA

- word단위 → 데이터를 묶은 단위인
    
    block단위 → word를 여러개 묶으면 block단위
    
    DMA는 block단위로 전송. 다만, system마다 크기가 다르다.