### UDP

흐름제어 미지원
오류제어 미지원
손상된 세그먼트 수신에 대한 재전송 미지원

포트들을 사용해 IP 프로토콜에 인터페이스를 제공하는 것

TCP 초기 설정에서 드는 비용보다 더 적은 메세지가 요구되는 프로토콜이다.
UDP를 사용하는 예시로는 DNS가 있다.



### TCP

데이터 전송의 신뢰성 + 패킷의 순차적인 전달이 필요함

신뢰성이 없는 인터넷을 통해 종단 간에 신뢰성 있는 바이트 스트림을 전송하도록 특별히 설계되었다.
TCP 서비스는 송신자와 수신자 모두 소켓이라는 종단점을 생성함으로써 이뤄진다.
TCP에서 연결 설정은 3-way handshake를 통해 이뤄진다.
(TCP 연결 해제가 4-way-handshake로 이뤄지는가에 대해서는 아직 확신하진 못함.)

모든 TCP 연결은 full-duplex, point-to-point 방식입니다.
full-duplex란 전송이 양방향으로 동시에 일어날 수 있음을 의미합니다.
point-to-point란 각 연결이 정확하게 2개의 종단점을 갖고 있음을 의미합니다.
TCP는 멀티캐스팅과 브로드캐스팅을 지원하지 않습니다.


