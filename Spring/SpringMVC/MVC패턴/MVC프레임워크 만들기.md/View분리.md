# View 분리

## 반복적인 JSP Forwarding부분을 처리

```
public class MyView {

    private String viewPath;

    public MyView(String viewPath){
        this.viewPath = viewPath;
    }

    public void render(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException{
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
}
```

view를 통해서 공통 코드였던 forwarding부분을 따로 빼서 한번만 작성하도록 한다.<br>

## 각 controller에서 활용

`new MyView("/WEB-INF/views/save-result.jsp");`처럼 url만 넘겨서 Myview를 반환해주고 FrontController에서 해당 인스턴스의 render method를 활용해서 dispatcher를 통해서 forward를 진행한다.<br>
공통되는 코드를 MyView를 통해서 관리하고 controller를 호출할 때만 요청된 url이 실행되어 코드가 작동할 수 있게끔할 수 있다.<br>

## 장점

FrontController에서 공통된 코드를 처리한다.<br>
즉, 각각의 controller에서 MyView(즉, forwarding내용)을 호출하는 것이 아니라 객체만 반환하고 이를 FrontController에서 render()를 호출함으로써 처리한다.<br> 그렇게 JSP forward가 일어나게 된다.<br>

**따라서 공통된 코드를 Controller마다 작성하는 과정과 처리하는 과정을 할 필요없고 url만 담아서 MyView에 넘겨주고 FrontController에서 render를 호출해서 forwarding을 통해서 처리하게 되는 것이다.**