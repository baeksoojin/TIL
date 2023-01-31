# PORT

> 인터넷 프로토콜에서 운영 체제 통신의 종단점을 포트라고 한다. 네트워크나 프로세스를 식별하기 위한 논리적 단위로 Transport Layer에서 사용된다.

- TCP/IP에서의 패킷
TCP/IP 패킷 안에 TCP의 역할은<br>
출발지 ip address + port, 목적지 ip address+port, 전송제어, 순서, 검증 정보<br>
를 포함함으로써 구현된다.<br>

같은 서버의 여러개의 서비스(사용자)가 존재할 때(게임, 쇼핑) 게임서버와의 연결을 위한 numA, 쇼핑을 위한 웹브라우저 numB port는 ip address는 동일하지만 portnumber가 달라서 구분이 가능하다.<br>

