# Model 추가

## 서블릿 종속성 제거

사실 controller는 RequestResponse가 아닌 파라미터 정보만 사용해도 된다.<br>
```
 @Override
public MyView process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    //비즈니스로직이 들어가는 controller
    String username = request.getParameter("username");
    int age = Integer.parseInt(request.getParameter("age"));
    ...
}
```
위에서는 사실 RequestResponse 자체보다는 요청 파라미터만 필요하다.<br>
이처럼 RequestResponse와 HttpServletRequest 자체가 불필요한데 호출되고 있는 것을 확인할 수 있다.<br>

`setAttrirube`를 사용해서 데이터를 저장하고 사용했던 로직을 model을 만들어서 관리한다.<br>


## 뷰 이름 중복 제거
controller는 뷰의 논리 이름을 반환하고 실제 물리 위치의 이름은 프론트 컨트롤러에서 처리하도록 단순화 할 수 있다.<br>
`/WEB-INF/views/[뷰이름].jsp`처럼 동일한 부분을 프론트 컨트롤러에서 처리한다.<br>
향후 뷰 폴더의 위치가 변경되더라도 프론트 컨트롤러만 변경하면 된다.<br>

## Frontcontroller에서의 Model 추가버전

1. FrontController의 컨트롤러 조회<br>
2. 요청받은 Controller를 컨트롤러 호출후 ModelView(model과 view 두가지를 포함하는 객체)를 반환<br>
3. viewResolver를 호출후 MyView를 반환<br>
4. render를 FrontController에서 MyView method를 통해서 forwarding


## ModelView(Model과 View를 분리해서 사용)

분리해서 사용하기 이전)<br>
- 지금까지 컨트롤러에서 서블릿에 종속적인 HttpServletRequest를 사용했다.<br>
- `request.setAttribute()`를 사용해서 데이터를 저장하고 뷰에 전달하였다.<br>
- 서블릿의 종속성을 제거하기 위해서 Model을 직접 만들고 View 이름까지 전달하는 객체를 만들어보자!

```
@WebServlet(name="frontControllerServletV3", urlPatterns = "/front-controller/v3/*")
public class FrontControllerServletV3 extends HttpServlet {

    private Map<String, ControllerV3>
            controllerMap = new HashMap<>();
    //url을 통해서 controller를 호출할거여서 key를 string, value를 컨트롤러

    public FrontControllerServletV3(){
        controllerMap.put("/front-controller/v3/members/new-form", new MemberFormControllerV3());
        controllerMap.put("/front-controller/v3/members/save", new MemberSaveControllerV3());
        controllerMap.put("/front-controller/v3/members", new MemberListControllerV3());

    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("FrontControllerServlet3.service");

        //map을 통해서 어떤 컨트롤러가 호출되었는지 찾고 jsp를 호출

        String requestURI = request.getRequestURI();

        ControllerV3 controller = controllerMap.get(requestURI);
        if(controller==null){
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        //model에 데이터를 저장하는 단계
        Map<String, String> paramMap = creteParamMap(request);

        //controller를 통해서 요청된 controller만 처리하는 단계
        ModelView mv = controller.process(paramMap);
        mv.getViewName();//논리만 얻어옴

        String viewName = mv.getViewName();//논리이름 new-form
        MyView view = viewResolver(viewName);//물리적으로 매핑

        view.render(mv.getModel(), request, response);
    }

    private static MyView viewResolver(String viewName) {
        return new MyView("/WEB-INF/views/" + viewName + ".jsp");
    }

    private static Map<String, String> creteParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();
        request.getParameterNames().asIterator().forEachRemaining(
                paramName -> paramMap.put(paramName, request.getParameter(paramName))
        );
        return paramMap;
    }
}
```

프론트 컨트롤러의 역할은 많아졌지만 실제로 구현할 Contoller는 매우 간소화되었고 중복된 코드를 없앨 수 있었다.<br>

ex) saveController

```
public class MemberSaveControllerV3 implements ControllerV3{

    private MemberRepository memberRepository = MemberRepository.getInstance();
    @Override
    public ModelView process(Map<String, String> paramMap) {
        String username = paramMap.get("username");
        int age = Integer.parseInt(paramMap.get("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        ModelView mv = new ModelView("save-result");
        mv.getModel().put("member",member);
        return mv;
    }
}


```

- `createParamMap()`<br>
HttpServletRequest에서 파라미터 정보를 꺼내 Map으로 변환해야한다. 그리고 Map이 paramMap을 컨트롤러에 전달하며 호출한다.<br>

- 뷰 리졸버<br>
`MyView view = viewResolver(viewName)`<br>
컨트롤러가 반환한 논리 뷰 이름을 실제 물리 뷰 경로로 변경한다.<br>
이후 실제 물리 경로가 있는 MyView 객체를 반환한다.<br>
