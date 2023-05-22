# Github branch로 협업하기

## Git branch란?

### 사전 지식

- 우선, Git은 변경사항이 어떻게 저장되어야하는지를 알아야한다.
    
    ⇒ Git의 변경사항은 “스냅샷”으로 저장이 된다.
    
    git에서 `commit`을 할 경우, 스냅샷에 대한 포인터 저자나 커밋 메시지 등과 같은 메타데이터, 이전 커밋에 대한 포인터 등을 포함하는 커밋 개체를 저장하게 된다.
    

- git commit 내용

1. 루트 디렉토리와 각 하위 디렉토리의 트리 개체를 체크섬과 함꼐 저장소에 저장.
2. 커밋 개체를 만들고 메타 데이터와 루트 디렉토리 트리 개체를 가리키는 포인터 정보를 커밋 개체에 넣어서 저장한다.

만약에 디렉토리 하나에 3가지의 파일이 존재하는데 initial commit을 한다면?

총 3개의 Blob과 파일과 디렉토리 구조가 들어있는 트리 개체 하나가 생성되고 메타 데이터와 루트 트리를 가리키는 포인터가 담긴 커밋 개체 하나로 총 5개가 생성된다.

루트 트리를 가리키는 포인터가 있는 커밋 개체가 `98ca9` 라면?

- 파일을 수정하고 커밋하면 이전 커밋이 무엇인지 역시 저장하게 된다.

ex) `98ca9` 에서 수정사항이 생겼고 역시 commit을 하면 루트 트리를 가리키는 포인터가 있는 커밋 개체가 생길텐데 id를 `34ac2` 라고 해보자.
기존 initial commit과 다르게 이전의 commit이 무엇인지도 저장하게 된다. 커밋 개체 안에 parent가 생기게 되는데 parent는 `98ca9` 를 가진다.

변경된 파일은 역시 snapshot로 기록이 되고 snapshot commit 개체에 blob로 변경 파일이 저장되어있을 것이다.


### 브랜치 만들기?

이때, 커밋과 커밋사이를 가볍게 이동하고 싶다면? 포인터같은 것을 사용하면 된다.

> 브랜치란? 커밋과 커밋 사이를 자유롭게 이동할 수 있는 포인터 같은 것이다.
> 

- 브랜치 생성
    
    ```
    git branch [branch이름]
    ```
    
    생성된 branch는 당장은 현재 작업 중이던 마지막 커밋을 가리키게 되어있다.
    
    따라서 위의 사진에서 작업이 멈춰있다면 `f30ab` 를 master와 새롭게 생성한 브랜치가 모두 가리키고 있을 것이다.
    
- branch 옮기기
    
    Git은 아직 HEAD(현재 작업 중이던 로컬 브랜치)가 master를 가리키고 있다.
    
    branch를 옮기기 위해서는 이동하는 명령을 내려줘야한다.
    
    ```
    git checkout [branch이름]
    ```
    
    그렇다면 현재 작업 진행을 가리키는 포인터인 HEAD가 이동한 branch를 가리킬 것이다.
    
- branch위에서 커밋하기
    
    branch로 작업 공간을 이동하고 수정 후에 커밋을 한다면?
    
    master는 기존에 가리키던 `f30ab` commit을 그대로 가리키고 있는다.
    
    반면, branch에서는 commit이 진행되었기 때문에 새로운 commit 개체가 `87ab2` 라면 그것을 가리킬 것이다.
    
- branch나가고 master에서 다시 작업하기
    
    master로 다시 이동을 하기 위해서 `git checkout master` 를 진행한다.
    
    그렇다면 HEAD는 master를 가리킬 것이고 워킹 디렉토리 파일도 해당 시점으로 변경되어 branch에서의 작업 내용은 보이지 않는 상태가 될 것이다.
    
    **branch 작업 내용은 별도로 진행되기 때문에 master branch로 돌아오면 확인할 수 없다.**
    
    수정사항이 생겼다면 master에서 커밋을 진행할 것이다. 커밋을 한다면, master의 앞에는 새로운 커밋 개체가 생성되었을 것(`c2b9e` 라고하자)이다. 그렇다면 지금 `f30ab` 를 가리키고 있는 것은 총 2개가 될 것이다.
    
    이때, 브랜치는 2개로 갈라지게 된다.
    
- log 활용
    
    ```
    git log --oneline --decorate --graph --all
    ```
    
    해당 명령어를 사용하면 log를 확인해서 branch 리스토리를 볼 수 있다.
    
- 생성과 checkout 동시에 하기
    
    ```
    git checkout -b [branch 이름]
    ```
    
- 합치기
    
    branch의 변경사항을 적용하기 위해서는 `merge` 를 사용한다.
    
    ```
    git merge [merge할 브랜치 이름]
    ```
    
- branch 삭제하기
    
    ```
    git branch -d [삭제할 브랜치 이름]
    ```
    

- merge conflict 해결하기
    1. 어디서 충돌이 진행됐는지를 알아야한다.
    
    ```
    git status
    ```
    
    ⇒ 어떤 파일을 Merge 할 수 없었는지 살펴본다.
    
    1. 충돌이 나는 부분을 없애고 코드를 수정하는 과정을 거친 후에 git add 명령어를 활용해 다시 git에 저장한다.
    2. 다시한번 체크!! → 제대로 고쳐졌고 충돌나는 부분이 존재하지 않는지 git status로 체크한다.

---

## remote branch

- remote는 리모트 저장소에 있는 포인터인 레퍼런스를 의미한다. 즉, 브랜치, 태그 등을 의미한다.
    - 형태 : `<remote>/<branch>` 로 remote 저장소를 먼저 적고 살펴보고 싶은 브랜치 이름을 적으면 branch 확인이 가능하다.
    - clone을 하면 origin이 만들어지게 될 것이고 master를 확인하고 싶다면? origin/master라는 이름을 사용하면 된다.

- 저장소 동기화
    - `git fetch origin` 명령을 사용하면 리모트 서버에서부터 저장소 정보를 동기화할 수 있다.
    - origin/master로 master pointer까지 존재하던 새로운 commit이 있다면 모두 내려받아서 “동기화를 “ 진행한다.
    - 내 PC에서 origin/master가 있었던 곳에서 fetch를 진행하면 upstream의 origin/master의 master가 존재하는 곳으로 origin/master pointer의 위치가 변경된다.
        
        이때, 기존에 작업을 진행하던 branch가 있었다면 2 way 처리가 되게 된다.
        
- 새로운 서버 저장소를 연결하기
    
    우선 새로운 서버 저장소를 `git remote add` 를 사용해서 **현재 작업 중인 저장소에 새로운 서버의 저장소를 추가를 할 수 있다.**
    
    추가를 하였다면 추가하려던 새로운 서버 저장소의 데이터를 내려받아야하는데 `git fetch [새로운 서버 저장소의 이름]` 과 같이 내려받을 수 있도록한다. 
    
    그렇다면, 새로운 서버 저장소의 master가 해당 서버의 이름을 따서 <새로운 서버의 이름><master>와 같이 포인터가 생길 것이다.
    
- push하기
    
    로컬의 브랜치를 서버로 전송하려면 쓰기 권한이 있는 리모트 저장소에 PUSH한다.
    
    ```
    git push <remote> <branch>
    ```
    
    원하는 서버이 remote 저장소에 branch의 commit을 추가하기 위한 명령어이다.
    
    이후, 누군가 저장소를 Fetch하고 서버에 있는 브랜치에 접근할 때 origin<branchname>으로 접근이 가능하다.
    
    이렇게 origin에 push가 되었다면?
    
    - fetch하기
    
    ```
    git fetch origin
    ```
    
    origin의 작업 내용중 변화된 부분을 모두 가져오게 된다.
    
    ⇒ 수정할 수 없는 리모트 트래킹 브랜치를 내려받게 된다.
    
    다음으로 merge를하거나 이어서 작업을 하는 경우 두가지를 선택해서 처리하면 된다.
    
    - merge하기
    
    ```
    git merge origin/<branch이름>
    ```
    
    - merge 하지 않고 리모트 트래킹 브랜치에서 시작하는 새 브랜치 만들기
    
    ```
    git checkout -b [branch이름] [origin/branch이름]
    ```
    
    위와 같이 작성하면 정한 branch이름을 가지며 origin/branch이름에서 시작하고 수정할 수 있는 로컬 브랜치가 만들어진다.
    
- branch 추적하기
    
    
    리모트 트래킹 브랜치를 로컬 브랜치로 Checkout을 하면 **Tracking branch**가 생성된다.
    
    → 트래킹 브랜치는 Upstream의 브랜치이다.
    
    - 트래킹 브랜치는 리모트 브랜치와 직접적인 연결고리가 있는 로컬 브랜치이기 때문에 데이터를 내려받을 때는 `git pull` 을 사용하다.

- PULL하기
    
    fetch를 진행하고 데이터를 불러오기 위해서 merge를 진행한다.
    
    - 일반적으로 `git pull` 보다는 `fetch` → `merge` 를 진행하는 것이 더 낫다.

---

## Git branch - Rebase하기

Git에서 한 브랜치에서 다른 브랜치를 합치며 협업을 진행하게 되는데, 두가지 방법이 있다.

두개의 차이점에 대해서 알아봐야한다.

- merge

merge는 두 브랜치의 마지막 커밋 두개와 공통 조상을 하나 더 사용하는 3-way Merge를 활용해서 새로운 커밋을 만든다.

- rebase

![image](https://github.com/baeksoojin/TIL/assets/74058047/4e71d9b5-f493-4517-8c81-2cdaf06efe32)

실제로 두 브랜치가 나뉘기 전인 공통 커밋으로 이동하고 (c2가 될 것임) 그 커밋부터 지금까지에서 checkout을 experiment로 했다면 checkout이 가리키는 커밋까지 diff를 만들어 어딘가에 저장을 해놓는다.

Rebase할 브랜치가 합칠 브랜치가 가리키는 커밋을 가리키게 하고 저장해놨던 변경사항을 차례대로 적용을 해준다. 다음으로 master 브랜치를 `Fast-Forward` 를 시키게 된다.

![image](https://github.com/baeksoojin/TIL/assets/74058047/11064cb3-cd9e-4477-91ae-b857343d081b)

- 차이점
    
    커밋 히스토리가 다르다.
    
    rebase를 사용하면 Merge commit이 남지 않아서(통합작업 필요없음) 더 깨끗한 히스토리를 만들 수 있다.
    
    rebase한 브랜치를 살펴보면 히스토리가 선형이 된다.
    
- 공통점
    
    최종 결과물은 같다.
    

### 주의사항

✅ 이미 공개 저장소에 push한 커밋을 rebase하지 마라

rebase는 기존의 커밋을 그대로 사용하는 것이 아니라 내용은 같지만, 다른 커밋을 새로 만든다.

⇒ 만약 잘못해서 엉망이 된다면 ****Rebase 한 것을 다시 Rebase 하기!****
