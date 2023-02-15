# 매우 간단한 상품 관리 웹사이트를 만들어보자

Spring이 우선이기에, thymeleaf와 html, css, bootstrap 관련 내용은 해당 글에서 다루지 않겠다.<br>

------
## 세팅

- Project : Gradle, Java, Spring Boot는 Test버전이 아닌 것으로 선택
- Project Metadata : *project name에는 -가 들어가면 안 된다.*, Jar를 사용해서 Tomcat 내장
- Dependency : Spring Web (MVC제공), Thymeleaf(template framework), Lombok(annotation제공)

---


##  요구사항 분석

관리자가 상품을 관리 할 수 있는 기능만 존재하는 아주 간단한 서비스를 생성하는 것을 요구사항으로 한다.<br>

- **Feature**<br>
등록한 "상품의 목록"이 보이도록한다.<br>
상품 목록에서 상품을 선택하면 "상품 상세" 내용이 보여야한다.<br>
상품을 "등록"할 수 있는 기능이 있어야한다.<br>
상품의 내용을 "수정"할 수 있는 기능이 있어야한다.<br>

- **Domain** : 상품ID,상품이름, 가격, 개수

- **main homepage**

    ![image](https://user-images.githubusercontent.com/74058047/219132572-9ef5443d-d84e-4b36-b980-a9208b3353d4.png)

- Service Logic Flow

    ![image](https://user-images.githubusercontent.com/74058047/219133863-dba19fef-abe1-4a74-ac6d-e27eaac7a4e8.png)

    **Controller를 거친 다음으로 Template이 호출되어야한다.**<br>
    
    <mark>몇가지 살펴볼 부분</mark><br>

    상품 저장 Controller가 실행된후, 렌더링하는 페이지는 상품 상세페이지<br>
    상품 수정 Controller는 변경된 내용을 Redirect를 통해 적용하고(상품 상세 Controller로 Redirect) 상품 상세 페이지를 렌더링<br>

-------

## 상품 도메인 개발

- Item

    ```
    @Data //다만 불필요한것까지 다 만들어서 위험함(원래는 주의할 필요가있음)
    public class Item {

        private long id;
        private String itemName;
        private Integer price;
        private Integer quantity;

        public Item(){
        }

        public Item( String itemName, Integer price, Integer quantity) {
            this.itemName = itemName;
            this.price = price;
            this.quantity = quantity;
        }


    }

    ```

- repository

    ```

    @Repository //component scan의 대상이됨
    public class ItemRepository {

        private static final Map<Long, Item> store = new ConcurrentHashMap<>();// 동기처리가 안 돼서 hashmap을 사용하면 안 됨
        private static long sequence = 0L;

        //save
        public Item save(Item item){
            item.setId(++sequence);
            store.put(item.getId(),item);
            return item;
        }

        //find
        public Item findById(Long id){
            return store.get(id);
        }

        //findall
        public List<Item> findAll(){
            return new ArrayList<>(store.values()); // 한번 감싸서 반환하면 arraylist에 값을 넣어도 store를 건드리는게 아니여서 store에는 변화가 없어서 안전성을 유지
        }

        //update
        public void update(Long itemId, Item updateParam){ //사실 해당 경우에는 Item을 사용하는 것이 아닌 사용하지 않는 id를 제거한 parameter3개만 사용하도록 DTO를 만드는 것이 맞음
            Item findItem = findById(itemId);
            findItem.setItemName(updateParam.getItemName());
            findItem.setPrice(updateParam.getPrice());
            findItem.setQuantity(updateParam.getQuantity());
        }

        public void clearStore(){
            store.clear();
            //test에서 사용예정
        }

    }

    ```

    * Repository의 역할<br>
    데이터를 처리하는 작업을 수행한다. 즉, **DB 접근을 하고 도메인 객체를 DB에 저장하고 관리하는 영역이다.** <br>
    
    - code check<br>
    1. spring container에 bean 등록을 위한 component scan<br>
        `@Repository` annotation을 사용
    2. Map을 활용해서 Domain에서 정의한 item을 관리<br>
        `private static final Map<Long, Item> store = new HashMap<>();`<br>
        store 인스턴스를 사용해서 id값과 item 객체를 key, value로 설정하여 사용.<br>
        save : store.push(item.getId(), item) / findById : store.get(id)/ findAll : store.values()

    3. clear
        test를 위해서 생성


-----

## Controller

### 컴포넌트 스캔과 Repository 의존성 주입
```
@Controller
@RequestMapping("/basic/items")
@RequiredArgsConstructor
public class BasicItemController {

    private final ItemRepository itemRepository;

    //controller 로직구현

}
```

- 코드설명<br>

    Controller => component scan
    RequestMapping("/basic/items") => 모든 url의 처음을 "basic/itmes"로 설정함
    RequiredArgsConstructor과 final => 생성자 주입
    

### 아이템 조회를 위한 controller

상품 상세 컨트롤러와 뷰가 필요하다.<br>
[모든 아이템 조회]
```
@GetMapping
public String items(Model model){
    List<Item> items = itemRepository.findAll();
    model.addAttribute("items",items);
    return "basic/items";
}
```

[상품 상세 페이지를 통한 상세 조회]
```
@GetMapping("/{itemId}")
public String item(@PathVariable Long itemId, Model model){
    Item item = itemRepository.findById(itemId);
    model.addAttribute("item",item);
    return "basic/item";
}
```

- method : 단순 조회이기에 GET을 사용
- parameter 처리 : @PathVariable을 사용해서 itemId로 저장
- repository : findById를 통해서 item 객체를 가져와서 model에 저장
- Model : model의 addAttribute로 key, value 쌍으로 렌더링할때 넘겨주고 thymeleaf에서 해당 key로 value에 접근하여 동적으로 html code를 변경
- Stirng type의 return : spring의 templates 하위의 basic/item 경로에있는 html code로 렌더링



## 아이템 등록을 위한 Controller

[등록 form]

```
@GetMapping("/add")
public String addForm(){
    return "basic/addForm";
}
```
form을 단순히 보여주면 되기에 Get method 사용

[등록수행]
```
//    @PostMapping("/add")
    public String addItemV1(@RequestParam String itemName,
                            @RequestParam int price,
                            @RequestParam Integer quantity,
                            Model model){
        Item item = new Item();
        item.setItemName(itemName);
        item.setPrice(price);
        item.setQuantity(quantity);

        itemRepository.save(item);

        model.addAttribute("item", item);

        return "basic/item";
    }

//    @PostMapping("/add")
    public String addItemV2(@ModelAttribute("item") Item item,Model model){

        itemRepository.save(item);
        return "basic/item";
    }


//    @PostMapping("/add")
    public String addItemV3(@ModelAttribute() Item item,Model model){

        itemRepository.save(item);
        return "basic/item";
    }//이름을 생략한 버전. name속성은 Item class의 첫글자를 소문자로 변경


//    @PostMapping("/add")
    public String addItemV4(Item item){
        itemRepository.save(item);
        return "basic/item";
    }//@ModelAttribute자체를 생략

    //@PostMapping("/add")
    public String addItemV5(Item item){
        itemRepository.save(item);
        return "redirect:/basic/items"+item.getId();
    }//@ModelAttribute자체를 생략

    @PostMapping("/add")
    public String addItemV6(Item item, RedirectAttributes redirectAttributes){
        Item savedItem = itemRepository.save(item);
        redirectAttributes.addAttribute("itemId",savedItem.getId());//url에 넘길것
        redirectAttributes.addAttribute("status",true);//query parameter로 넘기고 처리
        return "redirect:/basic/items/{itemId}"; //encoding을 처리해주는 redirectAttribute
    }//@ModelAttribute자체를 생략
```

**[version6(final version)설명]**<br>

- method => 등록을 위하여 POST<br>
- @ModelAttribute 생략 <br>
=> @ModelAttribute에 <mark>등록할 이름(Item class니까 첫글자를 소문자로 바꾼 item이 됨)이 addItemV6의 첫번째 인자에서의 item과 동일</mark>하기에 생략이 가능하고 <mark>class를 처리하고 있기에 굳이 적지 않아도 ModelAttribute로 spring이 처리해줘서 자체도 생략</mark><br>
- RedirectAttributes : html에서 올바르게 처리하기 위해 도입<br>
=> url의 parameter를 encoding해서 넘겨줌<br>
=> 처리 성공을 사용자가 알 수 있도록(사용자 경험을 긍정적으로 만들어야함) 하기 위해서 status를 true로 설정가능 -> thymeleaf에서는 queryparameter를 처리하는 문법을 지원한다<br>
`<h2 th:if="${param.status}" th:text="'저장 완료!'"></h2>`
- redirect<br>
detail page로 이동시키기 위해서 redirect를 사용

### 세부 내용을 수정

세부 내용이 어떤 것에 대한 것인지 id로 넘겨줄 필요가 있기에 url에는 parameter를 통해서 ID값이 들어와야한다.<br>

[수정 form]
```
@GetMapping("/{itemId}/edit")
public String editForm(@PathVariable Long itemId, Model model) {
    Item item = itemRepository.findById(itemId);
    model.addAttribute("item", item);
    return "basic/editForm";
}
```

code 설명
- method : 단순히 form을 렌더링하는 것이여서 GET method
- @PathVariable : parameter binding
- Model : item을 key로하고 item 객체를 value로 하여 모델에 저장
- String return type : templates안에서 해당 경로의 파일을 찾아서 model을 전달하고 동적 처리 이후에 client에게 전달


[edit 후 redirect to detail page]
```
@PostMapping("/{itemId}/edit")
public String edit(@PathVariable Long itemId, @ModelAttribute Item item) {
    itemRepository.update(itemId, item);
    return "redirect:/basic/items/{itemId}";
}
```

- method : form을 입력받아서 변경하는 것이기에 POST
- @PathVariable : parameter binding
- logic : item update를 실행
- ModelAttribute : form data를 읽어옴과 동시에 item 객체에 저장되게함
- repository : update를 사용하여 data를 update
- redirect : redirect:[원하는 이동 경로]를 redirect 경로로 설정하여 해당 경로의 로직을 처리하는 controller로 redirect됨.

tip) id값은 변경하지 않으니까 변경될 값만 처리하는 DTO를 만들어서 item class 대신 사용할 수 있다.<br>

-----

## 참고사항 및 추가개념정리

1. Data anootation<br>
    ![image](https://user-images.githubusercontent.com/74058047/219135567-b51e2c16-d4c8-4d26-82fb-a8f06d23fdd4.png)
    
    주의사항 : 불필요한 기능까지 사용할 수 있게 돼서 주의해서 사용해야함<br>
    EqualAndHashCode의 경우, equals 메소드와 hashcode 메소드를 생성하는데 객체를 저장한 뒤 필드값을 변경하면 hashcode가 변경되어 이전에 저장한 객체를 찾을 수 없는 문제가 발생한다.<br>
    추가로 RequiredAugsConstructor 역시 주의사항이 존재하는데 [여기](https://kwonnam.pe.kr/wiki/java/lombok/pitfall)를 들어가면 자세히 나와있다.<br>

    < DTO <br>
    단순히 데이터 교환에서 사용하기 위해서 만드는 DTO일때 정도까지만 @Data를 허용한다.<br>
    
    <사용> <br>
    왠만해서는 사용하지 않도록하고 @Getter, @Setter 처럼 **명시해서 사용하자.**

2. HashMap은 사용하지 말것 -> ConcurrentHashMap

    `private static final Map<Long, Item> store = new HashMap<>();`을 사용하면 동기처리가 안 되기 때문에 multi-thread 환경에서 오류 파티를 일으킴<br>
    따라서, `private static final Map<Long, Item> store = new ConcurrentHashMap<>();`로 고쳐서 사용해야한다.<br>

    ConcurrentHashMap의 경우 동기처리를 지원하여 멀티스래드 환경에서도 "안전하게" 작동하여서 `ConcurrentHashMap`을 사용한다.<br>

3. 동기

    ![image](https://user-images.githubusercontent.com/74058047/219142308-144f7b14-0812-4354-926f-a895b4e88310.png)
    thread가 모두 하는 일을 함께 신경쓴다면 synce

4. PRG(Post -> Redirect -> Get)<br>

    Post를 하고 Redirect를 하지 않고 그 상태에서 **새로고침**을 누른다면?
    <br>
    ![image](https://user-images.githubusercontent.com/74058047/219162084-1a38350d-5c09-463c-ac37-91c87d115b89.png)<br>

    새로고침 : "마지막"에 서버에 전송한 데이터를 다시 전송한다.<br>
    따라서 위의 사진처럼 post/add를 재실행한다.<br>

    **결제 페이지에서 결제 완료 후 그 위치에서 새로고침을 한 경우, 돈이 또 나가는 문제가 발생한다.**<br>

    <해결방법>
    <br>
    단순하다. 결과가 처리됨을 보여주는 화면으로(GET method를 사용중인 detail data를 확인할 수 있는 url로) Redirect를 진행한다.<br>
    ![image](https://user-images.githubusercontent.com/74058047/219164545-80befdcf-198d-4c28-875c-16fee68bf39e.png)