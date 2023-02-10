# HTTP 응답

#### HTTP 응답의 경우 크게 "3가지"로 분류가 가능하다.<br>
### 정적 리소스, 뷰 템플릿 사용, HTTP 메시지 사용의 경우를 살펴보자!

----
## 1. 정적 리소스

- 웹 브라우저에 정적인 HTML, css, image, js 등의 **정적 리소스**를 사용하는 경우<br>

- 디렉토리 경로<br>
    `/static` , `/public`, `/resources`, `META-INF/resources`
    - src/main/resources 안의 <span style="font-size:17px; background-color : blue; color:white;">static 하위의 자원은 내장 톰켓에 의해서 자동으로 서빙을 해준다.</span>
    - 예를 들어보자.<br>
    `src/main/resources/static/basic/hello-form.html`을 경로로하는 정적 리소스를 만들었다면 <span style="background-color : blue; color:white;">static 하위의 폴더의 경로부터 url에 넣어주면 static file을 server가 응답한다.</span><br>
    ```
    @RequestMapping("response-view-v1")
    public ModelAndView responseViewV1(){
        ModelAndView mav = new ModelAndView("response/hello")
                .addObject("data","hello!");
        return mav;
    }
    ```
    controller가 다음과 같을 때,<br>

    ![image](https://user-images.githubusercontent.com/74058047/218153953-3fa0582f-70cc-4fc4-906e-efa027307cb3.png)
    <br>해당 이미지처럼 경로가 설정되어있다면<br>
    <br>
    `localhost:8080/basic/hello-form.html`로 실행을 한다면 hello-form.html이 뜬다.<br>
    <img width="512" alt="image" src="https://user-images.githubusercontent.com/74058047/218155353-b9287c20-0c31-4de4-aa80-d1c51b4037f2.png"><br>
    hello-form.html 파일이 client로 잘 응답되는 것을 확인할 수 있다.<br>

----

## 2. 뷰 템플릿

- 동적으로 html을 생성하는데 사용한다. view가 응답을 만들어서 전달한다. html뿐만 아니라 다양한 형태로 렌더링이 가능하다.<br>

- 뷰 템플릿 경로<br>
    `src/main/resource/templates`
    - 동적 페이지는 <span style="font-size:17px; background-color : blue; color:white;">templates 하위의 경로에서 찾아서 렌더링해준다.</span>
    - 예를 들어보자<br>
    ```
    @Controller
    public class ResponseViewController {

        @RequestMapping("response-view-v1")
        public ModelAndView responseViewV1(){
        ModelAndView mav = new ModelAndView("response/hello")
                .addObject("data","hello!");
        return mav;
        }
    
    }

    ```
    `ModelAndView`를 통해서 "response/hello"를 넘겼다는 것은 `resource/templates/response/hello`경로의 페이지로 데이터를 넘기겠다는 것이다.<br>
    이때, 데이터는 `addObject`를 통해서 등록한 key인 data에 hello!가 들어간다.<br>
    ![image](https://user-images.githubusercontent.com/74058047/218160505-96938c79-39c0-45e3-9587-445210e2dc74.png) <br>
    이렇게 경로 설정을 하였기 때문에 `response/hello` 경로를 mav 인스턴스를 생성할 때 넘겨주는 것이다.<br>

    이때, html 코드가 아래와 같다면?<br>
    ```
    <html xml:th="http://www.thymeleaf.org">
    <body>
    <p th:text="${data}">empty</p>
    </body>
    </html>
    ```
    data에 저장된 value인 hello가 text에 들어가서 empty가 아닌 hello!가 출력된다.<br>
    ![image](https://user-images.githubusercontent.com/74058047/218161320-8e0e3725-e2ca-4889-a668-0de228a9bd3d.png)

- <span style="background-color : blue; color:white;">Controller에서의 String으로의 경로반환</span>

    `@Controller`를 사용하고 `return`값을 string으로 할 때, <span style="color : blue; background-color : white; font-weight:bold">그것이 html 경로가된다.</span>

    ```
    @RequestMapping("response-view-v2")
    public String responseViewV2(Model model){
        model.addAttribute("data","String을 활용한 template 경로설정");
        return "response/hello";
    }
    ```
    다음과 같이 작성이 가능하고 데이터를 저장하기 위해서 `Model interface`를 사용한다.<br>
    html code는 위의 예시에서 활용했던 것과 동일할 때 결과는 아래와 같다.<br>
    ![image](https://user-images.githubusercontent.com/74058047/218163073-db3e4265-3bed-40de-8417-d8f2f57f3d61.png)

    하지만, return값을 지정해주지 않고 싶다면 `@RequestMapping`로 넘기는 경로와 templates의 하위의 경로가 일치하면 그곳으로 응답데이터를 넘기고 데이터 처리 후에 client로 페이지를 렌더링해준다.<br>
    <span style="color : blue; background-color : white; font-weight:bold"> 다만 이 경우는 가시성이 너무 없어서 주로 사용하지 않는다.</span>

    ```
    @RequestMapping("/response/hello")
    public void responseViewV3(Model model){
        model.addAttribute("data","annotation과 templates폴더 하위 경로를 일치한경우");
    }
    ```
    ![image](https://user-images.githubusercontent.com/74058047/218165567-6f0d9b9b-278a-4f4a-a3ec-bb4a7be615dd.png)
    <br>결과가 `/response/hello`에서 잘 나오는 것을 확인할 수 있다.

----

## 3. <span style="color : black; background-color : #fff61b; font-weight:bold;">HTTP 메시지 사용 </span>

- `@ResponseBody`를 사용하여 Converter를 자동으로 실행한다.<br>
    - controller<br>
    ```
    @Controller
    public class ResponseBodyController {

        @ResponseStatus //ResponseEntity처럼 응답코드 사용을 위해 적용
        @ResponseBody //mesage converter사용
        @GetMapping("/response-body-json")
        public HelloData responseBodyJsonV1(){
            HelloData helloData = new HelloData();
            helloData.setUsername("userA");
            helloData.setAge(20);
            return helloData;
        }

    }
    ```
    - result<br>
    <img width="621" alt="image" src="https://user-images.githubusercontent.com/74058047/218170744-e4628821-fa32-43c1-991d-e080626e065d.png"><br>
    
- `@Controller`와 `@ResponseBody`를 합친 <span style="background-color : white; color : blue; font-weight:bold;">@RestController</span><br>
    매번 사용할 때마다 annoataion을 써주기 복잡하니 class의 annotation으로 @ResponseBody를 빼낸다면 그 문제를 해결할 수 있을 것이다.<br>
    이때, @Controller와 @ResponseBody를 합친 annoataion이 `@RestController`여서 이것을 사용하면 된다.<br>


    - controller<br>
    ```
    @RestController
    public class ResponseBodyController {

        @ResponseStatus //ResponseEntity처럼 응답코드 사용을 위해 적용
        //    @ResponseBody //mesage converter사용
            @GetMapping("/response-body-json2")
            public HelloData responseBodyJsonV2(){
                HelloData helloData = new HelloData();
                helloData.setUsername("userB");
                helloData.setAge(20);
                return helloData;
            }
    }
    ```

    - result<br>
    <img width="791" alt="image" src="https://user-images.githubusercontent.com/74058047/218172285-bc73526a-c084-4eac-b778-5c0489ee4955.png">

