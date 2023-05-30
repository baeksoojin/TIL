## Page Table Structure

### Hierarchical Page Tables

각 장에 대한 정보를 담고 그 안에 각 절에 대한 페이지를 구분. → 두가지 레벨이 존재해서 Two level page table이라고함.

Page를 찾아서 frame을 가져오고 그 안에서 원하는 것을 찾으면됨.

### Hashed Paged Tables

hash function을 무엇으로 결정하냐에 따라서 성능이 달라진다.

예를 들어서, 간단하게 주어진 주소값인 P를 10으로 나눈 나머지를 hash function이라고 하면 0~9까지가 나온다. (10개중에서 하나)

그렇다면 21,31,41,51 등등이 같은 곳(collision)으로 위치하게 된다. 그래서 포인터를 사용해서 연결해서 이어놓으면 계속 뒤로 넘어가면서 찾는 값이 나올 때까지 탐색해야한다.

hashing function을 사용하면 모두 탐색을 진행하지 않아도 된다는 점이 장점이지만 그 사이즈가 너무 작다면 연이어서 계속 연결된것을 탐색해야함.

### Shared Page

read-only code 를 공유해서 사용해서 이미 깔려져있으면 깔지 않는 것처럼 사용함.

(private code는 서로 다른 문서들)

아래한글 프로그램일 때 서로 공유하고 있으니 모두 같은 table number를 가지고 있다. 그러나 private 한 것은 당연히 가리키는 주소값이 다름(프레임의 저장 메모리의 주소값)

### Segmentation

- memory관리차원에서 paging방식(size가 모두 똑같아서 관리입장에서는 더 편리)과 segmentation(크기는 다 다르지만, 한 덩어리로 묶어서 사용자가 보기 편하게해서 사용자입장에서 더 편리)이 존재하는데 segmentation은 기계보다 사람에게 편리함.

- base, limit으로 처음과 끝으로 구분.
    - 시작의 위치를 base로 사용
    - length( → limit)로 어디까지인지 경계
    - 만약에 더 작다면, d값이 더해지면서 특정 segment의 위치를 찾아가게 됨.

**그 밖의 특징**

segment의 시작주소를 바꿔주면 위치가 변경되어 dynamic relocation이 가능.

아래한글과 같은 공유해서 사용하는 프로그램이 차지하는 segment가 있으면 sharing이 가능.

Allocation으로 first fit/best fit을 가질 수 있으며 external fragmentation의 문제가 발생할 수 있다.(그래서 compaction을 해서 한쪽으로 밀어버려야함)

### Segmentation Architecture Cont

read, write, execute privilege를 알려줘야지 구분이 가능.

sharing이가능하고 dynamic하게 사용이 가능하다.

### Segmentation with Paging

segmentation은 사용자에게만 편리하지 관리자(기계)의 입장에서는 편리하지 않아서 segmentation을 모두 쪼개서 paging처리해서 관리한다.

segmentation방식으로 시작(주소도 segmentation방식으로 지정 : base, limit과 함께)

찾으면 page-table의 시작주소를 보고 paging방식이 시작됨.

d값보다 작은 페이지 사이즈를 가지고 있는지 해보고 맞다면 페이지를 모두 쪼개서 p와(page table의 시작주소) d’을 만들어줌.

ex) 3번 segment가 page 5개를 필요로 하고 (사실은 여기저기 퍼져있는 것) pagination 적용