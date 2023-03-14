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

**Parameter Passing**

- Register

레지스터를 사용

- 전달되어야하는 parameter를 stack으로 처리

parameter를 push → 읽을때 pop

⇒ 두 경우는 사이즈를 미리 고정.

- Memory

일부분을 block 으로 잡아서 사용하면 일부 유동적으로 사이즈를 변화시킬 수 있음.

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