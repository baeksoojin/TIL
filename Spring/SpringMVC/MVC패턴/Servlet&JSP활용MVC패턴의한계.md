# MVC 패턴의 한계

> MVC 패턴은 model, view, controller로 나눠서 controller를 통해서 항상 먼저 거치고 view로 넘어가게끔 하는 패턴이다.

**servlet과 jsp만을 사용했을 때에는 모두 view와 controller의 역할이 구분되지 않는다는 문제점이 있다. 따라서 MVC패턴을 적용시키는 것에 대한 필요성이 확인된다.**
model, view, controller pattern을 의미하는데 controller를 통해서 service logic 관련 업무를 처리한다.

<MVC의 로직과 템플릿코드처리>
- 항상 contoller를 통해서 logic을 처리한다.<br>
- model에 담아서 데이터를 서로 주고 받는다.<br>
- view를 통해서 html 등을 처리한다.<br>

<JSP를 템플릿 코드를 통한 작성을 할 때>
WEB-INF를 통해서 controller를 통해야 항상 jsp가 실행될 수 있도록 하게 된다.<br>
redirect가 아닌 forwarding을 사용해서 client에서 html code를 재요청하는 방식이 아니라 server에서 처리하여 최종 html을 넘기는 방식을 사용한다.<br>

**다만 몇가지 한계점을 가진다.**

## 한계

- member 등록
```
@WebServlet(name="mvcMemberFormServlet", urlPatterns = "/servlet-mvc/members/new-form")
public class MvcMemberFormServlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        String viewPath = "/WEB-INF/views/new-form.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);//controller에서 view이동시에 사용
        dispatcher.forward(request,response);//servlet에서 jsp를 호출하는 것이 됨 server안에서 내부적인 작업으로 redirect의 개념이 아님.
    }
}
```

- save
```
@Override
protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

    //비즈니스로직이 들어가는 controller
    String username = req.getParameter("username");
    int age = Integer.parseInt(req.getParameter("age"));

    Member member = new Member(username, age);
    memberRepository.save(member);

    //model에 데이터 보관

    req.setAttribute("member", member);


    //view로 전환
    String viewPath = "/WEB-INF/views/save-result.jsp";
    RequestDispatcher dispatcher = req.getRequestDispatcher(viewPath);
    dispatcher.forward(req, resp);

}
```
위에서 data를 model에 담아서 dispathcer를 통해서 viewPath에 전송하는 것을 확인할 수 있다.<br>

- list 출력
```
@Override
protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

    //controller
    System.out.println("MvcMemberListServlet.service");
    List<Member> members = memberRepository.findAll();

    //model에 저장

    req.setAttribute("members", members);

    // view로 전달
    String viewPath = "/WEB-INF/views/members.jsp";
    RequestDispatcher dispatcher = req.getRequestDispatcher(viewPath);
    dispatcher.forward(req,resp);
}
```
보면 controller를 제외한 부분이 계속 반복되고 있는 것을 확인 할 수 있다.<br>

**한계점**<br>
**반복되는 코드가 많다. servlet&JSP로만 만들면 한계가 발생한다.**

- forward 중복<br>
controller에서 중복이 되게 많고 forward부분을 항상 반복하게 된다.<br>

- ViewPath의 중복<br>
확장자가 바뀐다면 ViewPath가 들어간 전체 코드를 모두 접근하여 바꿔줘야한다.<br>
`/WEB-INF/views`가 중복되고 `.jsp`가 중복된다. prefix&suffix가 중복되는데 다른 view로 변경하게 된다면 이를 모두 변경해야한다는 것이다.<br>

- 사용하지 않는 코드<br>
`service(HttpServletRequest req, HttpServletResponse resp)`를 호출하고 사용하지 않는다.<br>

- 공통 처리가 어렵다<br>
utils에서 공통 처리 메서드를 만들어도 그것을 결국 호출할때 호출자체에 대한 중복이 일어난다.<br>
해결하기 위해서는,<br>
servlet이 호출되기 전에 공통기능을 먼저 처리해야한다.<br> 항상 서블릿이 호출되기 전에 "문지기"역할을 하며 공통 처리가 먼저 일어나야한다.<br>

**front controller를 도입하여 수문장 역할을 하는 기능을 두어 컨트롤러 호출 전에 공통 기능을 한번에 처리해야한다. 입구를 하나로 만드는데 스프링 MVC의 핵심이 있다.**