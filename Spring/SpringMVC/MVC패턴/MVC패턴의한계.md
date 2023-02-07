# MVC 패턴의 한계

> MVC 패턴은 model, view, controller로 나눠서 controller를 통해서 항상 먼저 거치고 view로 넘어가게끔 하는 패턴이다.

model, view, controller pattern을 의미하는데 controller를 통해서 service logic 관련 업무를 처리한다.

<MVC의 로직과 템플릿코드처리>
- 항상 contoller를 통해서 logic을 처리한다.<br>
- model에 담아서 데이터를 서로 주고 받는다.<br>
- view를 통해서 html 등을 처리한다.<br>

WEB-INF를 통해서 controller를 통해야 항상 jsp가 실행될 수 있도록 하게 된다.<br>
redirect가 아닌 forwarding을 사용해서 client에서 html code를 재요청하는 방식이 아니라 server에서 처리하여 최종 html을 넘기는 방식을 사용한다.<br>

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


