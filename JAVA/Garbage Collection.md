# Garbage Collection

**GC**

> Automatic garbage collection is the process of looking at heap memory, identifying which objects are in use and which are not, and deleting the unused objects. - [oracle java GC Basic](https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html)


## GC가 필요한 이유[Why]

JAVA는 **Memory Fragmentation**의 성질을 가지고 있다. 이것이 문제인데 CPU의 cache memory로 이러한 fragemented인 memory가 사용되게 되기 때문이다.<br>
memory fragementation은 연속으로 저장되지 않고 쪼개져서(흩어져서) 저장될 수 있기에 이때, cache memory에 저장될 수 있다.<br>

<img width="688" alt="image" src="https://user-images.githubusercontent.com/74058047/219024791-3fef7413-faca-4283-88f4-a034852dede9.png">


이때,cache memory는 CPU에서 매우 빠르게 동작하며 비싸지만 이것을 unless data로 사용하게 되는 것이다. <br>
따라서 메모리 조각화의 문제점을 해결하기 위해서 java는 특히 성능이 매우 좋은 가비지 수집기(GC)가 필요한 것이다.<br>
<br>

**참고) Go에서는 필요하지 않은 이유**

go는 단일 포인터는 단일 할당으로 header-less 이지만, java는 poinnter의 개별할당이 필요하다. <br>
개별할당시에 "header"는 16바이트씩 오버헤드를 추가한다.<br>

- go에서는 추적할 객체가 1개
- java에서는 개별 개체를 모두 추적 및 관리.



-------


## Memory Fragmentation 극복을 위한 GC의 동작[How]

**Compaction** 
한 메모리 위치에서 다른 메모리 위치로 블록을 이동하여 메모리의 연속 블록을 수집하는 작업이 필요하다.<br>이를 Compaction이라고 한다.<br>
다만, 한 메모리 위치에서 다른 메모리 위치로 블록을 이동하려면 CPU 싸이클 비용이 필요하고 모든 참조를 새위치로 가르키게 하며 업데이트하는데도 비용이 필요하기에 비싸다.<br>

**stop-the-world**
이러한 업데이트를 수행하려면 "쓰레드 중지"가 필요하다.<br>
GC를 실행하기 위해서는 위에서 소개한 JVM이 애플리케이션 실행을 멈추는 'stop-the-world'가 발생한다.<br>
stop-the-world가 발생하면 GC를 실행하는 쓰레드를 제외한 나머지 쓰레드는 모두 작업을 멈추고 작업 후에야 다시 시작된다.<br>

따라서 이러한 긴 일시중지를 줄이기 위해서 Java는 **generational garbage collectors**이 필요하다.

###  generational garbage collectors 활용 Adding Complexity

1. grouping


    - 사용중인 Object<br>
    프로그램의 일부가 여전히 해당 개체에 대한 포인터를 유지하고 있음을 의미한다.<br> 사용중이지 않는 개체는 더이상 참조된 부분이 없음을 의미한다.<br>
    > Young objects — These have a low generation counter. Meaning they have only recently been allocated.

    - 사용중이지 않은 "참조되지 않은 객체"가 사용한 메모리를 회수할 수 있다.
    > Old objects — Objects that have survived multiple mark and sweep operations by the GC. A generation counter is updated on each mark and sweep to keep track of how old the objects are.

    ![image](https://user-images.githubusercontent.com/74058047/219028926-09647e99-a232-406c-b9d5-81d395b87a14.png)

2. Delet

    ![image](https://user-images.githubusercontent.com/74058047/219032202-a872fb18-4d25-420a-bb44-0244ae093e33.png)

    여유 공간에 대한 포인터를 남기고 참조되지 않은 개체를 제거한다.<br>

    - 압축을 통한 삭제

    ![image](https://user-images.githubusercontent.com/74058047/219032510-bf363a1b-64ad-4650-93c7-6e59412a77bc.png)

    compaction을 통해서 쪼개진 것을 압축해준다.<br>

- 필요한 이유<br>
    JVM이 모든 객체를 표시하고 압축하는 것은 매우 비효율적이기에 힙을 Generation화 해서 분리후에 압축을 통해서 삭제해야한다.<br>
    ![image](https://user-images.githubusercontent.com/74058047/219033349-545e4b65-cdc8-459a-9cdf-79714f44efdc.png)

- 결국 old generation을 모아서 삭제하는 과정으로 이루어져야한다.<br>

- 자세히 살펴보자면 아래의 과정으로 없어진다.<br>


1. Aging(사용중인 Obejct에 count값을 메기는 과정)

    ![image](https://user-images.githubusercontent.com/74058047/219034942-07c70a1b-f7b7-4390-846a-de129722e02d.png)
    Eden에 메모리가 할당되는데 다 찰 때 S0 -> S1로 사용되는 Object만 +1로 카운트값을 증가하여 옮긴다.<br>

    ![image](https://user-images.githubusercontent.com/74058047/219034964-d32a5845-7704-4267-9a13-c47704351bdf.png)
    위의 사진처럼 Eden 메모리가 할당될때 S1 -> S0로 사용되는 Object만 +1로 카운트값을 증가시킨다.<br>

이때, 특정 임계값이 도달하게 되면 older로 generate된다.
<br>

2. Promotion(Generation을 하는 과정)

    ![image](https://user-images.githubusercontent.com/74058047/219035716-f5de1427-ae59-4c9d-acea-b3529563ad82.png)

- 전체과정
    ![image](https://user-images.githubusercontent.com/74058047/219036511-29e4df00-3902-4c3a-8c47-73aa26ef49f4.png)


-----

## Old 영역에 대한 GC

- Serial GC

Old 영역에서 살아있는 객체를 식별하는 (MASK) 를 진행하고 힙의 앞부분부터 확인하며 살아있는 것만 남기는 Sweep을 한다.<br>
마지막 단계에서는 객체들이 연속되게끔 힙의 가장 앞 부분부터 채워서 존재하는 부분과 없는 부분으로 나눠 Compaction을 진행한다.
<br>

**적은 메모리와 CPU core 개수가 적을 때만 사용해야하고 그럴때 적합하다**

- Parallel GC

GC 처리하는 스레드가 하나가 아니라 여러개이다.<br>
병렬처리를 진행하여 serial GC보다 빠르게 처리할 수 있고 메모리가 충분하고 코어의 개수가 많을 때 유리하다.<br>
Throughput GC라고도 부른다.<br>

- Paralle Old GC

앞의 Parallel GC의 Sweep 과정 대신해서 Summary 단계를 거쳐서 "수행한 영역에 대해서 별도로 살아있는 객체를 식별한다는 덤에서 다르며 더 복잡한 과정을 거친다."

이외에도 CMS GC, G1 GC 등이 있다. 
![image](https://user-images.githubusercontent.com/74058047/219039593-5e1be3fc-cbb5-434b-b3e1-e6bc45a8562c.png)

G1 GC는 그 어떠한 방식보다 빨라서 성능이 좋다. Young, Old 개념을 적용하면 안 되고 바둑판에서 각 영역에 대한 객체를 할당하고 얼마나 사용중인지에 따라서 삭제를 진행하는 방법이다.<br>




----

## 의문점 및 주의사항<br>

- Garbage Collector를 통해서 쓰레기를 찾아 지우는 과정을 해야한다.<br>

    따라서, `System.gc()` 매서드를 호출하면 큰 성능 저하 문제가 발생해서 절대 사용하면 안 된다.<br>

- Old에서 Young을 참조하는 경우

    <img width="466" alt="image" src="https://user-images.githubusercontent.com/74058047/219037594-a06f8bd9-8a2a-4912-8686-9fcfc62b52b2.png">

    Card Table을 통해서 참조를 표시하고 삭제대상인지 판단하게 된다.<br>


----


## 참고

- https://itnext.io/go-does-not-need-a-java-style-gc-ac99b8d26c60

- https://d2.naver.com/helloworld/1329

- https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html