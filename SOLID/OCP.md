# OCP

## Open/closed principle 개방-폐쇄 원칙

소프으웨어 요소는 확장에는 열려있으나 변경에는 닫혀 있어야한다.

## 다형성
interface를 변경할 뿐, 기존 코드를 다르게 변경하지 않는 "다형성"을 활용한다.

## Spring과 OCP
```
public class MemberService{
    private MemberRepository memberReposity = new MemoryMemberRepository();
}
```
로 되어있던 MemberService의 memberRepository를 MemoryMemberRepository가 아닌 JdbcMemberRepository 객체로 바꾸고 싶다면?
```
public class MemberService{
    private MemberRepository memberReposity = new JdbcMemberRepository(); 
}
```
다음과 같이 작성하게 된다.<br>

- client의 code MemberService의 변경의 문제<br>
Client code가 변경될때ㅑ OCP가 깨짐.

- 해결방안<br>
spring container와 DI를 활용하여 해결할 수 있다.