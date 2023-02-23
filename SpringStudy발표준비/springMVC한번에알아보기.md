# SpringMVC 한번에 알아보기

--------

## servlet

servlet을 MVC pattern을 적용하지 않고 사용하게 된다면?
<br>
1. 반복되는 코드가 많다.<br>
    `dispatcher.forward(request, response)`가 항상 중복된다.<br>
2. ViewPath의 중복이 발생한다.<br>
    "WEB-INF/views"~".jsp" 등에서 중복이 발생하는데 다른 view로 경로를 변경하게 된다면 **모두 경로를 변경** 해줘야한다.<br>
3. 사용하지 않는 코드가 많다.<br>
    `service(HttpServletRequest req, HttpServletResponse resp)`를 통해서 req, resp를 모두 호출하지만 사용하지 않는 경우가 있다.<br>
4. 공통 처리가 어렵다.<br>
    공통되는 로직을 한번만 작성하여 이것을 필요할때만 사용하고 싶은데, 모든 로직을 다 작성해서 계속 처리된다. servlet이 호출되기 전에, 공통기능을 우선으로 처리할 필요가 있다.<br>

## FRONTCONTROLLER를 통한 MVC pattern 적용 아이디어

<img width="820" alt="image" src="https://user-images.githubusercontent.com/74058047/220824412-adc63b33-1bac-458c-904c-76de1274bc17.png">

Request 처리 : `FrontController`를 통해서 핸들러를 조회하고 핸들러 어댑터 목록에서 알맞은 핸들러를 불러온다. 불러온 핸들러로 request를 처리하는 등의 동작을 실행하게 되고 ModelView가 반환된다.<br>

Responose 처리 : handler를 거쳐서 만들어진 ModelView를 받아서 viewResolver를 호출하고 MyView가 반환된다. response를 넘겨야하는 파일을 확인했다면 model을 호출하여 rendering을 진행하여 응답을 제공한다.<br>


------------

## Spring에서의 MVC Pattern처리

<img width="822" alt="image" src="https://user-images.githubusercontent.com/74058047/220824909-24f5b241-13cc-41cc-b373-af9cbf325eb0.png">

### DispatcherSevlet<br>

DispatcherServlet -> FrameworkServlet -> HttpServletBean -> HttpServlet 구조로 되어있기에 HttpServlet을 상속받아서 만들어진 것이 DispatcherServlet이다.<br>
FrontController의 역할을 DispatcherSevelt을 맡는다.

<특징 알아보기><br>
    1. `urlpatterns="/"`에 대해서 서블릿으로 자동등록되면서 디폴드로 매핑된다.<br>
    2. servlet이 호출되면서 `service()`를 호출하고 스프링MVC는 부모인 `FrameworkServlet`에서 service()를 오버라이딩해뒀다.<br>
    service()를 시작으로 `doDispath()`가 호출되고 이는 <mark>핸들러 조회 -> 핸들러오댑터 조회 -> 핸들러 어댑터 실행 -> 핸들러 오댑터를 통해 핸들러를 실행하고 ModelAndView를 반환-> 뷰 리졸버를 통해서 뷰를 찾고 -> 뷰를 반환받은 다음 -> 뷰 렌더링</mark>을 하는 과정이 담겨있다.<br>

< Model ><br>

아이템을 조회하는 기능을 만들때의 예시를 통해서 보자.
```
@GetMapping
public String items(Model model){
    List<Item> items = itemRepository.findAll();
    model.addAttribute("items",items);
    return "basic/items";
}
```
위의 예시처럼,<br>
Model을 활용해서 데이터를 응답해준다.<br>
`model.addAttribute(key,value)`형식으로 작성이 가능하다. key를 활용해서 template code에서 value에 접근이 가능하다.<br>

< View ><br>

위에서 봤던 코드를 보면 `return "basic/items";`으로 되어있고 method의 return type이 `String`이다.<br> 뷰 리졸버를 통해서 가져온 basic/items의 html code를 client에게 제공한다.<br>


----------

## HTTP request 처리 및 response 반환

--------------

## Request 처리

### 요청 매핑 처리 과정

1. url mapping type
`@RequestMapping('/url')`로 url을 적는다. <br>
이때 메서드는 모두 허용되고 디폴트는 'GET'이다.<br>
🟧 **스프링에서 url/과 url은 사실은 다른 url이 되지만 같은 요청으로 판단하고 매핑을 진행한다.**<br>

2. method를 지정해주는 방법
`@RequestMapping("/url",method = RequestMethod.GET)` 다음과 같이 method를 함께 넘겨준다.<br>
@PostMapping
@PutMapping
@DeleteMapping
@PatchMapping 등과 같이 축약도 가능하다.<br>
이때는 value로 url을 지정해줘야한다.<br>
`@GetMapping(value = "/url")`

3. PathVariable<br>
url을 template화 해서 url안의 변수를 `PathVariable` 어노테이션과 함께 사용한다.
```
@GetMapping("/mapping/users/{userId}/orders/{orderId}")
```
```
public String mappingPath(@PathVariable("userId") String userId, @PathVariable Long
        orderId) {
    ~~~
}
```

그 밖에도 `params`를 통한 특정 파라미터가 있어야 호출되도록 하는 방식, `headers`를 통해 특정 헤더가 있어야 동작하는 방식으로 조건을 걸 수도 있다.<br>
이외에도 `consumes`와  `produces`를 통해서 각각 서버, 클라이언트가 받아들일 수 있는 형태를 지정할 수 있는 기능도 있다.

자세한 내용은 [여기](https://github.com/baeksoojin/TIL/blob/main/Spring/SpringMVC/springmvc/%EA%B8%B0%EB%B3%B8%EA%B8%B0%EB%8A%A5/request/%EC%9A%94%EC%B2%AD%EB%A7%A4%ED%95%91.md) 에서 참고가능하다.<br>

### request Body를 통해서 넘어온 데이터 처리과정

🟩 springVMC에서 HttpServletRequest, HttpSevletResponse 보다 가볍게 원하는 것만 파라미터로 넘길 수 있도록 제공해준다. 공식문서를 들어가서 원하는 파라미터의 지원이 가능한지 확인하면서 작성하면 된다.<br>

Converter를 사용해서 Body의 내용을 조회할 때, 1. httpEntity를 사용하는 방법과 2. @RequestBody annotation을 사용하는 방법이 있다.<br>

### @RequestBody annotation

1. text일때

```
@ResponseBody
@PostMapping("/request-body-string")
public String requestBodyString(@RequestBody String messageBody) {}
```

2. Json이 넘어올때
```
@ResponseBody
@PostMapping("/request-body-json")
public String requestBodyJson(@RequestBody HelloData data) {
    log.info("username={}, age={}", data.getUsername(), data.getAge());
    return "ok";
}
```


이외에도 body를 통해서 들어오지 않고 form을 통해 들어온 경우에는 `getParam`을 통해서 처리할 수 있다. 또한 `@RequestParam`을 통해서 단순타입의 파라미터를 처리할 수 있으며 `@ModelAttribute`를 통해서 요청 파라미터를 받아서 객체에 자동 저장해주는 어노테이션을 활용해서 넘어온 파라미터를 처리할 수 있다.<br>

---------

## Response 처리

### 반환 타입

정적 리소스, 뷰 템플릿, HTTP message를 사용한 방법이 존재하는데 HTTP Message를 사용한 방법을 알아보자.<br>

### Http Message를 사용한 방법

```
@RestController
public class ResponseBodyController {

    @ResponseStatus //ResponseEntity처럼 응답코드 사용을 위해 적용
    @GetMapping("/response-body-json")
    public HelloData responseBodyJson(){
        HelloData helloData = new HelloData();
        helloData.setUsername("userB");
        helloData.setAge(20);
        return helloData;
    }
}
```

맞는 코드일까? 그렇다.<br>
마지막에 return이 helloData이다.<br>틀린것이 아닐까?생각할 수 있다.<br>
객체를 json으로 변환하는 과정을 거쳐야하는 것이 빠졌다고 생각할 수 있겠지만 anotation을 활용했다.<br>
response에서의 body converter는 `@ResponseBody`가 없어서 변환이 안 된다고 생각할 수 있겠지만, `@RestController`를 사용했기에 Object를 json으로 자동변환해준다.<br>

`@ResponseBody`가 동작한 것인데 이것은 Http message를 response를 반환하기 위해서 작성하는 경우 사용하는 annotation이다.<br>
이는 Converter의 기능을 가지고 있고 따라서 html file 등으로 반환하지 않기에 뷰 리졸버를 호출하지 않는다.<br>

<동작원리>
1. request
2. controller호출
3. @ResponseBody의 발견하고 return값을 체크함과 동시에 아래과정진행
4. HttpMessageConverter가 동시에 동작하면서 JsonConverter 혹은 StringConverter가 실행된다.<br>

<조건>
1. 대상 클래스 타입 지원 여부<br>
2. Accpet type의 지원 여부<br>


--------

☑️ hashmap을 사용하면 실무에서는 Multi-thread 환경에서 동작하기 어려움이 있을 수 있는데 그 이유는?<br>
그렇다면 무엇을 사용해야하나?<br>

☑️ 웹 브라우저에서 HTTP Message를 처리하기 위해서 TCP/IP listen부터 시작해서 socket통신 종료를 하는데 이때의 발생하는 모든 과정을 직접 코드로 작성하지 않고 대신 [_______]을 사용하면 된다.<br>
다만, 제대로 작성하지 않으면 코드에서 불필요한 부분이 있거나 중복이 많아 유지보수가 어려워질 수 있기에 [______]를 둬서 MVC pattern을 적용해야한다.<br>

☑️ POST를 통해서 client가 특정 요청을 넘겼다. 해당 페이지에 머물러있어도 될까?<br>
안된다면 어떻게 해야할까?<br>

☑️ Controller, Service, Repository, Domain 중에서 request url을 mapping하고 request data의 return type을 정해주는 곳은 어디일까? <br>

☑️ 아래에서 리팩토링을 해야하는 곳이 있다면?

```
public class Item {

    private long id;
    private String itemName;
    private Integer price;
    private Integer quantity;
}
```
```
 //update
public void update(Long itemId, Item updateParam){ 
    Item findItem = findById(itemId);
    findItem.setItemName(updateParam.getItemName());
    findItem.setPrice(updateParam.getPrice());
    findItem.setQuantity(updateParam.getQuantity());
}
```