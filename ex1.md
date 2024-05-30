# 유닉스 도메인 소켓 예제

유닉스 도메인 소켓은 같은 시스템에서 통신이 일어나므로 TCP/IP 프로토콜을 직접 이용할 필요가 없다. </br>
따라서 유닉스 도메인 소켓에서 사용하는 소켓 주소 구조체의 항목도 IP 주소가 아닌 경로명을 지정하도록 되어 있다. </br>
이는 파이프나 시스템 V IPC에서 특수 파일을 통신에 사용하는 것과 같다고 생각하면 된다. </br>
소켓 주소 구조체의 항목이 다른 것을 제외하면 유닉스 도메인 소켓이든 인터넷 소켓이든 소켓 함수를 사용하는 방식은 동일하다.

ex1이 나타내는 서버 프로그램은 "hbsocket"이라는 이름으로 소켓을 생성하고 클라이언트의 접속을 기다리다가 클라이언트가 접속하면 보내온 메시지를 읽어 출력하는 구조다.