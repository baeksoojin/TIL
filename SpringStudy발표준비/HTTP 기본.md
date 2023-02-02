# HTTP 기본

## 인터넷 네트워크

네이버에서 검색엔진을 활용한다고 해보자! 그 절차는 어떻게 될까?
네이버는 TCP/IP 프로토콜을 사용한다. 

1. DNS(domain name systen)
internet protocol address + port number로 구성된 ip를 naver.com과 같은 domain name을 활용하여 domain server에서 관리한다.<br>
사용자는 따라서 DNS를 활용해서 ip address를 불러와서 naver homepare에 접근이 가능하게 된다.<br>

2. client는 server에게 웹 화면을 요구한다[request] <br>
socket 통신을 통해서 packet을 전달하게 되는데 packet에는 sender address, receiver address, port number 등이 포함된다. 이때 TCP/IP는 순서를 보장하기 때문에 접근제어, 순서 등의 내용도 packet에 포함된다.<br>
이 과정에서 Three handshake가 발생하고 ack를 서로 주고 받게 된다.

3. server는 client에게 html 정보가 담긴 데이터를 제공한다[response]
이 과정도 역시 socket통신을 통해서 패킷이 전송되고 이때 최종적으로 receiver인 server도 sender인 client에게 ack를 넘기면서 마무리된다.<br>

4. 이렇게 받은 데이터를 parsing해서 client는 웹 페이지를 띄운다.<br>

--------

이렇게 인터넷 상에서 패킷이 전송되는 과정에서 "HTTP 통신"이 일어나게 되고 이때의 상태코드와 메서드들이 존재한다.