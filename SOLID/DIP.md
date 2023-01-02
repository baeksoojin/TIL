# DIP

## dependency inversion principle

> 프로그래머는 추상화에 의존해야지, 구체화에 의존하면 안 된다.

client는 interface에만 의존해야지 구체화된 구현 클래스에 의존하면 안 된다.

## OCP의 인터페이스 및 구현 클래스 동시 의존의 경우

```
MemberRepository m = new MemoryMemberRepository();
```
일때 MemoryMemberRepository가 interface에 의존하여 만들어져 interface에 의존하고 있는 것은 맞지만 클라이언트가 구현 클래스를 직접 선택함으로써 구현 클래스에도 의존하고 있다.

- 해당 경우는, *DIP를 위반한다*<br>
결론, 객체지향의 "다형성"만으로는 OCP와 DIP를 위반한다. 인터페이스를 통해서 다중 상속으로 역할과 그 역할의 구현체는 분리하여 설계하였지만 구현체를 client에서 바꿔 끼울 때, client code를 수정해야하는 부분이 생긴다.

- **Spring을 사용하여 DIP위반의 경우를 해결해야한다.**
