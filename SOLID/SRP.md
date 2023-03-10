# SRP

## Single reponsibility principle로 "단일 책임 원칙"

> 한 클래스는 하나의 책임만 가져야한다.

## 책임
단일 책임에서 책임이란 하나의 특정 class 혹은 module 등의 actor를 위한 기능 집합으로 클 수도 있고 작을 수도 있다.

**하나의 책임을 진다는 것이 어떤 의미인지 이해해보자!**

예를 들어서 개발자 class가 있다고 할 때, 개발자는 기획 및 디자인의 역할을 하면 안 된다는 것이다. 개발자 class안에서 기획을 정의해버리면, 기획안이 정의된 개발 class code를 모두 수정해야한다. 따라서 개발 class에는 개발의 내용만 정의하고 기획 class는 따로 만들어서 기획안을 수정해야할 것이다.

**변경**의 관점에서 생각해야한다.
책임지는 것이 많을수록 서로 의존도가 상승하게 되고 객체 지향적 코드 작성이 되지 않는다. 하나를 변경할 때 여러개가 결합되어있기에 그 여러개를 모두 변경해줘야한다는 문제점이 발생한다. 따라서 하나의 클래스는 하나의 책임만 가져야한다.