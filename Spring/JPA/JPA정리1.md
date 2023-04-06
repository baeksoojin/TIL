# JPA 기초 총정리

> 본 내용은 아직 json api를 살펴보기 전에 jpa에 대한 개념을 익히는 내용에 대한 글이다.
> 

- 애플리케이션 구조를 살펴본다.
- 도메인 분석 설계를 알아본다.
- 웹 계층 개발에 대해서 적용해본다.

---

## 1. 애플리케이션 구조

- application 아키텍처 구조
    
    **controller → service → repository → db(domain)**
    
    domain의 경우 controller, service, repository에서 모두 사용됨.
    
    - controller = web ⇒ 웹게층(service를 참조해서 웹에서 필요한 페이지를 라우팅)
    - service ⇒ 비즈니스 로직을 처리. **transaction** 단위로 처리 ⇒ test case를 만들어야함.
    - repository ⇒ JPA를 직접 사용하는 계층으로 **EM**으로 method를 등록
    - domain ⇒ **엔티티가** 모여있는 계층으로 모든 계층에서 사용됨.
    
- package structure

**<main>**

java > project

- projectApplication(@SpringBootApplication annotation → tomcat실행)
    - domain
    - repository
    - service
    - controller(+ post처리를 위한 form)
    - exception

java > resources

- static (css, js, index.html)
- templates (html)
- **application.yaml → properties로도 가능**

**<test>** 

---

- member에 대한 아키텍처 구조
    - domain
    
    ```java
    
    @Entity
    @Getter @Setter
    public class Member {
    
        @Id @GeneratedValue
        @Column(name = "number_id")
        private Long id;
    
        private String name;
    
        @Embedded
        private Address address;
    
        @OneToMany(mappedBy ="member")
        private List<Order> orders = new ArrayList<>();
    
    }
    ```
    
    <entity CLASS 설계>
    
    1. @Entity annotation을 통하여 entity 설계임을 알려줌. spring의 하이버네이트가 DB의 table과 entity를 연결해줄 것임.
    2. @Id, @GeneratedValue, @Column annotation, @Embedded, @OneToMany를 사용.
    
    database에서 사용할 table을 class로 만들어주고 annotation을 활용하여 spring에서 자동으로 table을 만들 수 있도록 설계함.
    
    ⇒ 모든 계층에서 불러서 사용할 예정.
    
    - repository
    
    repository에서는 “JPA”가 직접적으로 사용되는 단계로 Entity Manager를 사용하여 data의 crud를 관리한다.
    
    ```java
    
    @Repository
    @RequiredArgsConstructor
    public class MemberRepository {
    
        private final EntityManager em;
    
        public void save(Member member){
            em.persist(member);
        }
    
        public Member findOne(Long id){
            return em.find(Member.class, id); //반환타입과 id를 넘김
        }
    
        public List<Member> findAll(){
            return em.createQuery("select m from Member m", Member.class).getResultList();
        }
    
        public List<Member> findByName(String name){
            return em.createQuery("select m from Member m where m.name = :name",Member.class)
                    .setParameter("name", name)
                    .getResultList();
        }
    
    }
    ```
    
    - member와 관련된 crud가 어떤 것들이 일어나는지 먼저 설계한 후에 method를 만드는 단계이다.
    
    저장, 조회 등 각각 알맞는 메서드들을 만들어준다. 
    
    어떤 Input으로 어떤 Output을 결과로 내는지 보고 method를 짜면 된다.
    
    entity manager를 통해서 domain에서 정의한 member를 저장할 `save()`를 하나 살펴보면 member 객체 자체를 받아서 그것을 **transaction단위**로 저장할 수 있도록 entity manager의 영상태로 등록한다.
    
    - service
    
    만들어놓은 repository method를 활용하여 서비스 로직을 만드는 역할을 하고 transaction단위로 처리한다. 이때, 생성 및 업데이트 등이 발생하지 않고 단순 조회만 한다면 transactional annotation을 통해서 readonly처리를 해준다.
    
    ```java
    
    @Service
    @Transactional(readOnly = true)
    @RequiredArgsConstructor //final 이용 생성자 만들기
    public class MemberService {
    
        private final MemberRepository memberRepository; //생성자를 통해서 세팅 후에 변경할 일이 없음 -> final로 설정하기 : 컴파일 시점에 잡아주는 장점도 있음
    
        /**
         * 회원가입
         */
        @Transactional
        public Long join(Member member){
            validateDuplicatemember(member);
            memberRepository.save(member);
            return member.getId();
        }//등록
    
        private void validateDuplicatemember(Member member){
            //exception
            List<Member> findMember = memberRepository.findByName(member.getName());
            if (!findMember.isEmpty()) {
                throw new IllegalStateException("이미 존재하는 회원입니다.");
            }
    
        }
    
        //회원 전체 조회
        public List<Member> findMembers(){
            return memberRepository.findAll();
        }//조회
    
        //회원 단건 조회
        public Member findOne(Long memberId){
            return memberRepository.findOne(memberId);
        }//조회
    
    }
    ```
    
    단순하게 조회하는 것은 사실 위의 예제에서는 바로 service를 거치지 않고 **repository로 가도 무방한 상태이다.** 
    
    회원가입 로직처럼 멤버를 저장하는 method를 사용하는데 그 과정에서 `validateDuplicatemember`와 같이 유효성 검증을 하는 function을 추가해서 사용하는 로직을 짜는 단계이다.
    
    여기서도 역시, 어떤 input data를 가지고 어떤 output data를 controller에서 사용하게 될지를 고려하면서 작성해야한다.
    
    그렇다면 회원가입, 회원 조회 등의 서비스를 만들어 놨다면 이러한 서비스를 웹의 어떤 페이지에서 어떤 데이터를 입력받아서 사용하게 되는지 그 틀을 만들어줘야하는데 service를 웹에서 활용할 수 있도록하는 controller를 작성해야한다.
    
    <이 사이!!>
    
    controller를 작성하기 전에 service가 제대로 동작하고 예외처리 등의 로직이 먹히고 있는지 파악하기 위해서는 “**testcode**”를 잘 작성해야한다.
    
    testcode는 따로 살펴보는 것으로 하자.
    
    - controller
    
    ```java
    
    @Controller
    @RequiredArgsConstructor
    public class MemberController {
    
        private final MemberService memberService; //DI
    
        @GetMapping("/members/new")
        public String createForm(Model model){
            model.addAttribute("memberForm", new MemberForm()); // 빈 객체라도 가져가는 것은 validation이라도 체크해주기 때문
            return "members/createMemberForm";
    
        }
    
        @PostMapping("/members/new")
        public String create(@Valid MemberForm memberForm, BindingResult result){
    
            if(result.hasErrors()){
                return "members/createMemberForm";
            }
    
            Address address = new Address(memberForm.getCity(), memberForm.getStreet(), memberForm.getZipcode());
    
            Member member = new Member();
            member.setName(memberForm.getName());
            member.setAddress(address);
    
            memberService.join(member);
            return "redirect:/";
        }
    
        @GetMapping("/members")
        public String list(Model model){
            List<Member> members = memberService.findMembers();
            model.addAttribute("members", members);
            return "members/memberlist";
        }
    
    }
    ```
    
    - url mapping 방법을 선택하여 page를 rendering or redirect로 연결.
        
        parameter를 받고 domain객체를 불러와 setter로 등록하고 있는 것을 확인이 가능하다. 등록된 class object 자체를 service단에서 사용하고 회원가입 및 조회 등의 기능이 수행될 수 있도록한다. 
        
        Get ⇒ 프론트엔드에서 화면에 뿌려질 데이터를 가져오거나 화면을 가져오기 위해서 사용하는 method이다.
        
        POST ⇒ FORM 데이터와 같이, frontend에서 작성된 data를 백엔드로 넘겨서 사용해야할때 사용하는 method이다.
        

---

## 2. 도메인 분석 설계에 대해 알아보자

> 요구사항을 분석하고 그에 맞는 도메인 모델과 테이블을 설계하여 엔티티 클래스 개발하는 설계 방법이다.
> 

- 어떤것인지 알아보자.
- 엔티티 설계시의 주의점이 존재하는데 도메인 분석 설계를 해보며 알아보자.

- 요구사항 분석
    - 회원기능 → 회원 등록, 회원 수정
    - 상품 기능 → 상품 등록, 수정, 조회
    - 주문 기능 → 상품 주문, 주문 내역 조회, 주문 취소
    - 기타 요구사항 → 상품 제고 관리(주문과 연관), 종류관리, 상품 카테고리 구분, 배송 정보 입력

- 도메인 모델과 테이블 설계
    - entity domain model 설계
        - **객체에서는** 카테고리가 아이템의 리스트를 가져도 되고 아이템이 카테고리의 리스트를 가져서 서로 **다대다** 참조를 하도록 해도 되지만 **관계형 DB에서는 그렇게 해서는 안 되고 중간에 1:다 , 다 :1 로 연결하도록 중간에 mapping table**을 하나 만들어야함.
    - entity table 설계
        
        entity table을 설계할 때, 상속 관계에 있던 domain class들을 상속받는 부모 domain으로 합치는 과정을 통해서 single table mapping이 가능함. → 이때, 상속받은 class들을 구분하기 위해서 Dtype과 같은 구분자가 필요하긴하지만 성능이 좋음.
        
        - **다대다 관계는 거의 사용되지 않아야함.**
        
        실무에서 다대다를 사용해서는 안 됨
        

**<key point>**

항상 연관관계 매핑시 알아야할 점은 **“주인”은 FK가 있는 쪽이다.**

예를 들어서 자동차와 바퀴가 있을 때, 자동차 1개당 바퀴가 여러개. 따라서 바퀴는 어떤 자동차의 것인지 자동차의 id를 가지고 있음 → 일대다 관계에서는 “다” 쪽이 “주인”이 됨. 이처럼 두 객체의 연관관계중 FK 키를 가지고 관리하는 연관관계가 연관관계의 주인이 된다.

- 엔티티 클래스 개발

실제로 코드를 작성하는 부분이 된다.

- 설계시 주의사항 및 알아둘점
    - setter를 가급적 사용하지 말아야함.
        - 변경 포인트가 많아지면 유지보수가 어려움
    - **연관관계는 지연로딩으로 설정해야한다. (매우중요)**
        - **즉시로딩은 엮여있는 것들을 한번에 로딩하는 것**
            - **하나를 가져오면 연관된 것을 db를 다 끌어옴**
            - **특회, jpql로 하면 우선 egar를 무시하고 sql query로 하나 날라가는데 그때 100개를 가져왔으면 egar를 이때 체크하고 다시 들어가서 100개에 대한 쿼리를 또 날림 → n+1(query하나가 n개의 query를 또 만듦)의 문제가 발생**
            - **manytone는 기본이 egar로 설정되어있어서 주의해야함**
        
        **⇒ EAGER는 예측이 어렵고 어떤 sql이 실행될지 추척이 어려움.**
        
    
    ⇒ “모든 연관관계는” LAZY로 지연로딩으로 설정해야됨.
    
    - **컬렉션은 필드에서 바로 초기화**
    
    하이버네이트가 jpa로 영속성을 사용하면 감싸서 원하는대로 관리하는 클래쓰로 바꿈. 
    
    그래서 컬렉션은 필드에서 바로 초기화해야지 다른곳에서 꺼내서 초기화하거나 다시 꺼내쓰거나 하면 안 됨. → 만약에 다시 초기화되거나 다시 꺼내쓴다면 하이버네이트가 관리하는 영속성 컨텍스트를 잃게 된다.
    
    잘된예)
    
    ```java
    private List<Order> orders = new ArrayList<Order>();
    ```
    
    collection은 onetomany, manytomany 등의 한쪽에서 여러개의 값을 가질 수 있을 때 사용한다. 
    
    - `cascade`를 통해서 등록할때 안의 속성을 따로 등록하는 코드를 작성하지 않아도 알아서 등록해줌
    - 양방향 참조가 있을 때 편의 메서드 등록
        
        양방향 메소드가 있을 때, 하나를 등록하고 하나를 또 불러와서 그걸 셋해줘야하는데 이를 편리하게 하기 위한 메소드를 등록.
        

---

## Entity Class 개발

- **상속관계 매핑**

    전략을 singletable로 가져갈 때 **상속을** 해야하기 때문에 **추상 class**로 만들고 테이블에서 상속을 시켜줌.

    ⇒ `abstract class`와 `extends`를 통해서 상속이 가능

    ⇒ annotation 에서 @DiscriminatorValue를  통해서 dtype을 자식 table에 설정해줘야함.

    ⇒ annotation으로 @`DiscriminatorColumn` 을 통해서 (name=”dtype”)으로 설정해서 dtype을 불러와서 자식 class(entity)를 구분함.

- 부모 entity ⇒ strategy를 singletable

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE) //전략중 싱글테이블을 선택해서 다 때려?넣어줌
@DiscriminatorColumn(name="dtype")
@Getter @Setter
public abstract class Item {

    @Id
    @GeneratedValue
    @Column(name="item_id")
    private Long id;

    private String name;
    private int price;
    private int stockQuantity;

}
```

ex) Entity : Item에 book. album, movie가 있다면

```java
@Entity
@DiscriminatorValue("A")
@Getter
@Setter
public class Album extends Item {

    private String artist;
    private String ect;
}
```

```java
@Entity
@DiscriminatorValue("B")
@Getter
@Setter
public class Book extends Item{

    private String author;
    private String isbn;
}
```

```java
@Entity
@DiscriminatorValue("M")
@Getter @Setter
public class Movie extends Item{

    private String director;
    private String actor;

}
```

table)

- **일대일 관계일 때**

    일대일 관계에서는 FK를 아무데나 놔도 되지만 주로 **조회를 많이 하는 곳에 넣음.**

- **Getter, Setter**

    실무에서 조회할 일이 많아서 getter를 열어두는 것은 편함.

    다만, setter로 데이터가 변해지기 때문에 많이 열어두면 어디서 호출돼서 수정되는지 확인이 잘 되지 않음. 언제 어느타이밍에 변경되는지 나중에 코드 수정을 할 때 찾아야함.

    ⇒ `entity를 변경할 때. 변경용 비즈니스 메서드를 활용해야 유지보수가 효율적으로 가능함.`

- **객체와 table의 차이를 이해해야함**

    객체에서 table이름을 설정해주고, 연결관계를 설정해주고 manytomany일 경우, 객체는 가능하지만 table에서는 안 되기때문에 중간 다리를 매핑해주는 annotation을 만들어줘야한다는 점 등을 고려해야함.

- **연관관계 매핑(1:다 관계)**

```java
@OneToMany(mappedby = "many table에 있는 매핑된 컬럼") //일대다 관계로 해당 컬럼 하나에 여러개의 다른 컬럼을 가지는 경우
@ManyToOne // 다대일 관계로 여러개의 값이 하나의 컬럼에 들어갈 수 있는 경우. => 이때 해당 테이블이 "다"의 관계를 가지기에 FK를 가지고 있어서 JoinColumn으로 정의해주기
```

entity 2개가 있을때 class로 정의된 entity안에 다른 entity를 정의했다면 감싸고 있는 entity를 처음으로 생각해서 그거를 기준으로 하나일때 안에있는 entity가 여러개가 가능한지 등으로 연관관계를 적어주면 됨.

```java
@OneToMany(mappedBy ="member")
private List<Order> orders = new ArrayList<>();
```

```java
@ManyToOne(fetch= LAZY) //FK
@JoinColumn(name="member_id")
private Member member;
```

둘중에 하나를 업데이트를 해야할 때 → 테이블구조에서 FK를 확인해서 → 연관관계 주인을 찾아야함. 

“업데이터”시 주인 테이블만 업데이트함. 다른 테이블은 읽기전용으로 참조만 할 뿐 업데이트를 하지는 않음 → 따라서 이것을 알려주기 위해서 **`mappedby`** =””를 적어서 업데이트 용이 아니라 참조용으로 **읽기전용임**을 표시해줌.

`@JoinColumn` 을 사용해서 FK가 있고 **연관관계 주인임**을 보여주기

---

## Entity 설계시 참고사항

- 객체와 테이블의 차이
    - 연관관계 매핑방법
        - 객체는 참조로 연관관계를 맺음 → 양방향이 가능한 이유는 단방향 2개를 사용해서 양방향 참조처럼 보이는것임
        - 테이블은 fk로 연관관계를 맺음
    - 방향성
        - 참조는 단방향
        - fk는 join으로 양방향

---

## 3. 웹계층 개발

- **domain model pattern이란?**

entity에 핵심 비즈니스 로직을 몰아넣어서 처리하는 스타일로 생성로직을 entity에서 처리한다.

⇒ jpa ORM을 사용하면 보통 domain model pattern을 사용한다.

<참고>

- transaction script pattern이란?
    
    서비스 계층에서 대부분의 비즈니스 로직을 처리하고 엔티티에는 비즈니스 로직이 거의 존재하지 않는다.
    

- controller, service, repository, entity(domain)을 만들어서 처리하면 된다.
- 동적쿼리의 복잡성은 querydsl로 해결한다.

---

## 웹 계층 개발을 method에 따라서 살펴보자

회원, 주문 관련 서비스를 개발한다고 해보자.

여러 메서드가 있지만, post, get만 사용하면 충분하기에 POST, GET의 경우 위주로 알아본다.

- post
1. form page (Getmapping)
2. 사용자가 submit button을 클릭
3. form data class를 활용해서 넘어온 데이터 처리(PostMapping)
4. Post ⇒ Redirect ⇒ Get 처리를 진행해서 오류를 방지

회원등록 예제를 살펴보자

```java
@Getter @Setter
public class MemberForm {

    @NotEmpty(message = "회원 이름은 필수 입니다")
    private String name;
    private String city;
    private String street;
    private String zipcode;
}
```

```java
@PostMapping("/members/new")
public String create(@Valid MemberForm memberForm, BindingResult result){

    if(result.hasErrors()){
        return "members/createMemberForm";
    }

    Address address = new Address(memberForm.getCity(), memberForm.getStreet(), memberForm.getZipcode());

    Member member = new Member();
    member.setName(memberForm.getName());
    member.setAddress(address);

    memberService.join(member);
    return "redirect:/";
}
```

- get
1. 단순 조회이기에, (GetMapping) 사용
2. model 객체에 전송할 데이터를 담아 넘겨주면 됨.

예시로 주문리스트를 조회하는 코드를 작성해보자.

```java
@GetMapping("/orders")
public String orderlist(@ModelAttribute("orderSearch")OrderSearch orderSearch, Model model){
    List<Order> orders = orderService.findOrders(orderSearch);
    model.addAttribute("orders",orders);

    return "order/orderList";
}
```

지금까지 post, get 메서드 활용방법을 알아보았다. 이를 활용해서 상품 목록을 수정하는 코드를 기능을 개발해보자.

### 상품수정

1. 상품 목록에서 수정할 것일 선택 → id와 함께 get action
    
    ```java
    @GetMapping("items/{itemId}/edit")
    public String updateItemForm(@PathVariable("itemId") Long itemId, Model model){
    	 Book item = (Book) itemService.findOne(itemId);
    	
    	 BookForm form = new BookForm(); //entity가 아닌 form으로 전송하기
    	  form.setId(item.getId());
    	  form.setName(item.getName());
    	  form.setPrice(item.getPrice());
    	  form.setStockQuantity(item.getStockQuantity());
    	  form.setAuthor(item.getAuthor());
    	  form.setIsbn(item.getIsbn());
    	
    	  model.addAttribute("form",form);
    	  return "items/updateItemForm";
    
    }
    ```
    
2. form에 데이터를 입력해서 → id와 함께 post action
    
    ```java
    @PostMapping("items/{itemId}/edit")
    public String updateItem(@PathVariable Long itemId, @ModelAttribute("form") BookForm form){
    
    //        Book book = new Book();
    //        book.setId(form.getId());//준영속 entity로 영속성 부여를 받지 않음
    //        book.setName(form.getName());
    //        book.setPrice(form.getPrice());
    //        book.setStockQuantity(form.getStockQuantity());
    //        book.setIsbn(form.getIsbn());
    //        book.setAuthor(form.getAuthor());
            //itemService.saveItem(book);
    
            itemService.updateItem(itemId, form.getName(), form.getPrice(), form.getStockQuantity());
    
            return "redirect:/items";
    
        }
    ```
    
    - service
        
        ```java
        @Transactional
        public Item updateItem(Long itemId, String name, int price, int stockQuantity){
            Item findItem = itemRepository.findOne(itemId);
        		//findItem에 영속성이 생김 -> 변경 감지를 통해 자동으로 commit된후 flush되어 적용됨
            findItem.setPrice(price);
            findItem.setName(name);
            findItem.setStockQuantity(stockQuantity);
        
            return findItem;
        }//변경감지를 사용하는 방법 -> null update 가능성을 없앰
        ```
        
    
    수정 로직에서 중요한점은 merge가 아닌 영속성을 부여해서 transaction단위에서 **변경감지를 이용해서 commit시에 변경된 부분만 update**하는 것이다.
    

리팩토링 체크사항

- 위의 코드에서는 바로 Entity를 사용해서 request, response처리를 진행중인데 사실상 이렇게 하면 안 되고 **DTO를 만들어서 처리**해야할 필요가 있다.
- set method가 많을 때는 다른 메서드로 아예 빼버려서 처리한다.
    - ex)
    
    ```java
    @PostMapping("/items/new")
    public String create(BookForm bookForm){
        Book book = new Book();
        book.setName(bookForm.getName());
        book.setPrice(bookForm.getPrice());
        book.setStockQuantity(bookForm.getStockQuantity());
    
        book.setAuthor(bookForm.getAuthor());
        book.setIsbn(bookForm.getIsbn());
    
        itemService.saveItem(book);
        return "redirect:/";
    }
    ```
    

---

## 웹 계층 개발시 알아두면 좋은 점 및 주의사항

- entity자체를 사용하지 않고 DTO를 만들어서 사용한다.
- 수정시, **변경감지**를 사용해야한다.
    
    merge는 모든 속성이 변경되어 수정 form의 값에 null이있었다면 null로 update.
    
    변경감지는 **수정된 부분만 update.**
    
- cascade 사용시 주의사항

✅ cascade 사용 주의점

delivery, orderItem  두개는 모두 order에서만 사용한다. 
따라서 cascade를 사용해도된다. 
다만, 다른곳에서도 참조가 일어난다면 다른곳에 의해서 영향을 받을 수 있기에 주의해야한다.