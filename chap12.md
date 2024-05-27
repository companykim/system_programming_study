# chapter 12 소켓 프로그래밍 기초

TCP/IP 프로토콜은 계층구조로 구성되어 있으며, 5개 계층으로 이루어져 있다.

응용 계층: 사용자에게 서비스를 제공하기 위한 계층이다. 응용 계층의 대표적인 프로토콜은 Telnet, FTP, HTTP, SMTP, DNS 등이다.

전송 계층: 패킷의 전송을 담당하는 계층이다. 전송 계층의 대표적인 프로토콜에는 TCP와 UDP가 있다.

네트워크 계층: 인터넷 계층이라고도 하며 패킷이 전달되는 경로를 담당한다. TCP/IP에서 IP가 네트워크 계층에 속한다. IP 외에 ICMP, ARP 등이 있다.

네트워크 접속 계층, 하드웨어 계층: 물리적인 네트워크와의 연결을 담당한다. 일반적으로 이더넷 카드나 랜 카드라고 하는 부분이 이 계층에 해당한다.

TCP와 UDP의 차이점
| TCP | UDP |
| :--: | :--: |
| 연결지향형 | 비연결형 |
| 신뢰성 보장 | 신뢰성을 보장하지 않음 |
| 흐름 제어 기능 제공 | 흐름 제어 기능 없음 |
| 순서 보장 | 순서를 보장하지 않음 |

TCP </br>
전화를 걸 때처럼 데이터를 주고 받기 전에 송신측과 수신측이 연결되어 있어야 한다. 그리고 송신한 데이터가 </br>
수신측에 도착했는지 확인하는 과정을 거친다. </br>
또한, 데이터를 주고 받는 속도를 조절해 통신 효율이 떨어지거나 데이터가 손실되는 일을 방지할 수도 있다.

UDP </br>
TCP와 달리 사전에 목적지와 연결을 설정하지 않고 상대방이 데이터를 제대로 수신했는지도 확인하지 않는다. 단순히 목적지 주소로 지정해 네트워크로 전송하는 것이다. </br>
중도에 데이터가 분실되어도 신경 쓰지 않는다. 따라서 신뢰성보다는 속도가 중요한 서비스에 주로 이용한다.

TCP/IP 프로토콜을 이용해 데이터를 주고 받으려면 주소가 있어야 한다. 일반 우편으로 편지를 보낼 때 주소를 적는 것과 마찬가지다. </br>
TCP/IP 프로토콜에서 주소 관련 프로토콜은 IP다. </br>
TCP/IP 프로토콜을 이용하는 네트워크에서 주소는 IP 주소를 의미하며 IPv4에서는 점(.)으로 구분된 4바이트 정수로 표시한다.

IP 주소는 데이터가 전송될 목적지 호스트를 알려주는 역할을 한다. 그러나 목적지 호스트에는 여러 기능을 수행하는 프로세스들이 동시에 동작하고 있을 수 있다. </br>
따라서 전송되어오는 데이터를 어느 프로세스가 수신해 서비스를 제공할 것인지를 알려줘야 한다. 이때 사용하는 것이 포트 번호이다. </br>
포트 번호는 2바이트 정수로 되어 있으므로 0~65535까지 사용할 수 있다.

TCP/IP 프로토콜을 이용해 응용 프로그램을 작성할 때 TCP 계층에서 제공하는 인터페이스 함수를 직접 사용해 프로그래밍하면 매우 복잡하고 관련 프로토콜의 내부 구조를 알아야 한다. </br>
그것을 간편하게 해주는 것이 소켓 인터페이스다. 소켓 인터페이스는 응용 계층에서 전송 계층의 기능을 사용할 수 있도록 제공하는 응용 프로그래밍 인터페이스다. </br>
소켓 인터페이스를 이용하면 전송 계층이나 네트워크 계층의 복잡한 구조를 몰라도 네트워크 프로그램을 쉽게 작성할 수 있다.

## 1. IP 주소와 포트 번호

  TCP/IP 프로토콜을 이용해 통신하려면 IP주소가 있어야 하며, 인터넷에서 동작하는 각종 서비스를 구분하기 위한 포트 번호를 지정해야 한다.

  ### IP 주소와 호스트명
    IP주소는 인터넷을 이용할 때 사용하는 주소로 네트워크 주소(인터넷 주소)라고도 한다. 점(.)으로 구분된 32비트 숫자로 표시함.
    시스템에는 IP주소 외에 호스트명을 지정한다. 호스트명은 '호스트명 + 도메인명' 형태로 구성된다.
    도메인명은 도메인을 관리하는 기관에 등록하고 사용하고, 호스트명은 같은 도메인 안에서 중복되지 않게 시스템 관리자가 정해서 사용한다.

  ### 호스트명과 IP 주소 변환
    호스트명과 IP 주소를 등록해 놓은 파일이나 데이터베이스를 검색하면 호스트명이나 IP 주소를 찾을 수 있다.

    hosts: files dns -> 호스트명과 IP 주소를 먼저 파일에서 찾고, 파일에서 찾지 못하면 DNS 서비스를 이용한다는 뜻이다.

    호스트명과 IP 주소 읽어오기: gethostent(), sethostent(), endhostent()
    ---------------------------------------------
    #include <netdb.h>

    struct hostent *gethostent(void);
    void sethostent(int stayopen);
    void endhostent(void);

    * stayopen: IP 주소 데이터베이스를 열어둘 것인지 여부를 나타내는 값
    ---------------------------------------------
    호스트명과 IP 주소를 차례로 읽어온다.
    gethostent()는 호스트명과 IP 주소를 읽어서 hostent 구조체에 저장하고 그 주소를 리턴함.
    sethostent()는 데이터베이스의 현재 읽기 위치를 시작 부분으로 재설정한다. gethostent()를 처음 사용하기 전에 호출해야 함.
    sethostent() 함수의 인자인 stayopen 값이 0이 아니면 데이터베이스가 열린 채로 둔다.
    endhostent() 함수는 데이터베이스를 닫는다. 데이터베이스의 끝을 만나면 널을 리턴함.

    struct hostent {
      char    *h_name; 호스트명을 저장.
      char    **h_aliases; 호스트를 가리키는 다른 이름들을 저장함. 
      int    *h_addrtype; 호스트 주소의 형식을 저장. 
      int      h_length; 주소의 길이를 저장.
      char    *h_addr_list; 해당 호스트의 주소 목록을 저장함.
    };

    gethostbyname(): 호스트명으로 정보 검색
    ---------------------------------------------
    #include <netdb.h>

    struct hostent *gethostbyname(const char *name);

    * name: 검색하려는 호스트명
    ---------------------------------------------
    호스트명을 인자로 받아 데이터베이스에서 해당 항목을 검색해 hostent 구조체에 저장하고 그 주소를 리턴함.

    gethostbyaddr(): IP 주소로 정보 검색
    ---------------------------------------------
    #include <netdb.h>
    #include <sys/socket.h>

    struct hostent *gethostbyaddr(const void *addr, socklen_t len, int type);

    * addr: 검색하려는 IP 주소
    * len: addr 길이
    * type: IP 주소 형식
    ---------------------------------------------
    IP 주소를 인자로 받아 데이터베이스에서 해당 항목을 검색해 hostent 구조체에 저장하고 그 주소를 리턴함.
    
    type에서 사용하는 주소 형식
    AF_UNIX: 호스트 내부 통신
    AF_LOCAL: AF_UNIX와 동일
    AF_INET: IPv4 인터넷 프로토콜
    AF_AX25: 아마추어 라디오 AX.25 프로토콜
    AF_IPX: 노벨 프로토콜(IPX)
    AF_APPLETALK: 애플토크 프로토콜
    AF_X25: ITU-T X.25 / ISO-8208 프로토콜
    AF_INET6: IPv6 인터넷 프로토콜
    AF_KEY: 보안 키 관리 프로토콜(IPsec)
    AF_PACKET: 저수준 패킷 인터페이스
    AF_RDS: 신뢰할 수 있는 데이터그램 소켓(RDS) 프로토콜
    AF_PPPOX: 일반적인 PPP 전송 계층(L2TP, PPPoE)
    AF_LLC: 논리적 링크 제어 프로토콜(IEEE 802.2 LLC)
    AF_IB: 인피니밴드 주소
    AF_MPLS: 멀티프로토콜 레이블 스위칭
    AF_CAN: 컨트롤러 영역 네트워크 자동 버스 프로토콜
    AF_TIPC: 클러스터 도메인 소켓 프로토콜
    AF_BLUETOOTH: 블루투스 저수준 소켓 프로토콜
    AF_ALG: 터널 암호 API 인터페이스
    AF_VSOCK: VMware VSockets 프로토콜
    AF_KCM: 커널 연결 멀티플렉서 인터페이스
    AF_XDP: XDP(express data path) 인터페이스

  ### 포트 번호
    IP 주소는 데이터가 전송될 목적지 호스트를 알려주는 역할을 함. 목적지 호스트에는 여러가지 기능을 수행하는 서비스 프로세스들이 동시에 동작하고 있을 수 있다.
    그래서 전송되어오는 데이터를 어느 서비스 프로세스에 전달할 것인지 구분할 수 있어야 한다.
    인터넷에서도 IP 주소 외에 서비스를 구분하는 다른 정보가 필요한데 이 때 사용하는게 포트 번호다.
    포트 번호는 2바이트 정수로 되어 있으므로 0~65535까지 사용할 수 있다.

    포트 정보 읽어오기: getservent(), setservent(), endservent()
    ---------------------------------------------
    #include <netdb.h>

    struct servent *getservent(void);
    void setservent(int stayopen);
    void endservent(void);

    * stayopen: 포트 정보 데이터베이스를 열어둘 것인지 여부를 나타내는 값
    ---------------------------------------------
    포트 정보를 차례로 읽어온다.
    getservent() 함수는 포트 정보를 읽어서 servent 구조체에 저장하고 그 주소를 리턴한다.
    setservent() 함수는 데이터베이스의 현재 읽기 위치를 시작 부분으로 재설정한다. getservent() 함수를 처음 사용하기 전에 호출해야 한다.
    endservent() 함수는 데이터베이스를 닫는다.

    struct servent {
      char    *s_name; 포트명을 저장
      char    **s_aliases; 해당 서비스를 가리키는 다른 이름들을 저장한다.
      int     s_port; 포트 번호를 저장
      char    *s_proto; 서비스에 사용하는 프로토콜의 종류를 나타낸다.
    };

    getservbyname(): 서비스명으로 정보 검색
    ---------------------------------------------
    #include <netdb.h>

    struct servent *getservbyname(const char *name, const char *proto);

    * name: 검색할 포트명
    * proto: tcp 또는 udp
    ---------------------------------------------
    포트명을 인자로 받아 데이터베이스에서 해당 항목을 검색해 servent 구조체에 저장하고 그 주소를 리턴한다.

    getservbyport(): 포트 번호로 정보 검색
    ---------------------------------------------
    #include <netdb.h>

    struct servent *getservbyport(int port, const char *proto);

    * port: 검색할 포트 번호
    * proto: tcp 또는 udp
    ---------------------------------------------
    포트 번호를 인자로 받아 데이터베이스에서 해당 항목을 검색해 servent 구조체에 저장하고 그 주소를 리턴한다.

## 2. 소켓 프로그래밍

  소켓은 응용 계층과 전송 계층을 연결하는 기능을 제공하는 프로그래밍 인터페이스다.

  ### 소켓의 종류
    소켓은 [같은 호스트에서 프로세스 사이의 통신할 때 사용하는 유닉스 도메인 소켓]과 [인터넷을 통해 다른 호스트와 통신할 때 사용하는 인터넷 소켓]이 있다.
    이 소켓들을 표시하는 이름으로 아래와 같은 주소 패밀리명을 사용함.
    AF_UNIX: 유닉스 도메인 소켓
    AF_INET: 인터넷 소켓

  ### 소켓의 통신 방식
    소켓을 이용할 때도 하부 프로토콜로 TCP를 사용할 것인지 UDP를 사용할 것인지 지정해야 한다. 미리 정의되어 있는 상수를 사용함.
    SOCKET_STREAM: TCP 사용
    SOCKET_DGRAM: UDP 사용

    소켓의 종류와 통신 방식에 따른 네가지 통신 유형
    AF_UNIX - SOCK_STREAM
    AF_UNIX - SOCK_DGRAM
    AF_INET - SOCK_STREAM
    AF_INET - SOCK_DGRAM

  ### 소켓 주소 구조체
    소켓 구조체는 유닉스 도메인 소켓과 인터넷 소켓에서 각기 다른 형태를 사용함.

    유닉스 도메인 소켓의 주소 구조체
    struct sockaddr_un {
      __kernel_sa_family_t    sun_family;    /* AF_UNIX */
      char                    sum_path[UNIX_PATH_MAX]; /* 경로명 */
    };
    sun_family에는 AF_UNIX를 지정함.

    인터넷 소켓의 주소 구조체
    struct docksddr_in {
      __kernel_sa_family_t    sin_family; /* 주소 패밀리명 */
      __be16    sin_port; /* 포트 번호 */
      struct in_addr;    sin_addr; /* 인터넷 주소 */
    };

    struct in_addr {
      __be32    s_addr; /* 32비트 주소 */
    };

  ### 바이트 순서 함수
    컴퓨터에서 정수를 저장하는 방식은 각각 바이트를 순서대로 저장하는 빅 엔디언(big endian) 방식과 거꾸로 저장하는 리틀 엔디언(little endian)이 있다.

    빅 엔디언은 메모리의 낮은 주소에 정수의 첫 바이트를 위치시킨다.
    리틀 엔디언은 메모리의 높은 주소에 정수의 첫 바이트를 위치시킨다.

    컴퓨터마다 바이트를 저장하는 순서가 다르기 때문에 네트워크를 이용한 통신에서는 바이트 순서가 중요한 문제가 됨.
    데이터를 보내는 컴퓨터와 받는 컴퓨터의 정수 저장 방식이 다르면 같은 값을 서로 다르게 해석하기 때문.
    그래서 TCP/IP에서는 데이터를 전송할 때 무조건 빅 엔디언을 사용하기로 결정했는데, 이를 네트워크 바이트 순서(NBO)라고 한다.

    호스트에서 사용하는 바이트 순서는 호스트 바이트 순서(HBO)라고 한다. 시스템에서 통신을 통해 데이터를 내보낼 때는
    HBO에서 NBO로 데이터 순서를 바꿔서 전송하고, 데이터를 받으면 NBO에서 HBO로 변환한 후 처리해야 한다.

    --------------------------------------
    #include <arpa/inet.h>

    uint32_t htonl(uint32_t hostlong);
    uint16_t htons(uint16_t hostshort);
    uint32_t ntohl(unit32_t netlong);
    unit16_t ntohs(uint16_t netshort);

    * hostlong, hostshort: 호스트 바이트 순서로 저장된 값
    * netlong, netshort: 네트워크 바이트 순서로 저장된 값
    --------------------------------------
    htonl() 함수는 32비트 HBO를 32비트 NBO로 변환한다. htons() 함수는 16비트 HBO를 16비트 NBO로 변환한다.
    ntohl() 함수는 32비트 NBO를 32비트 HBO로 변환한다. ntohs() 함수는 16비트 NBO를 16비트 HBO로 변환한다.

  ### IP 주소 변환 함수
    IP주소는 점(.)으로 구분된다. 저장하는 방법은 두가진데, 시스템 내부적으로는 앞에서와 같은 형태의 주소를 이진값으로 바꿔서 저장한다.
    시스템 외부적으로 사용할 때는 문자열로 사용한다. 그래서 이진값과 문자열로 표시되는 IP 주소를 서로 변환할 수 있는 함수를 제공함.

    inet_addr(): 문자열 IP 주소를 숫자로 변환
    --------------------------------------
    #include <sys/socket.h>
    #include <netinet.h>
    #include <arpa/inet.h>

    in_addr_t inet_addr(const char *cp);

    * cp: 문자열 형태의 IP 주소
    --------------------------------------
    IP 주소를 문자열로 받아 이를 이진값으로 바꿔서 리턴한다. in_addr_t는 long형이다.

    inet_ntoa(): 구조체 IP 주소를 문자열로 변환
    --------------------------------------
    #include <sys/socket.h>
    #include <netinet/in.h>
    #include <arpa/inet.h>

    char *inet_ntoa(const struct in_addr in);

    * in: in_addr 구조체 형태의 IP 주소
    --------------------------------------
    IP 주소를 in_addr 구조체 형태로 받아 점으로 구분된 문자열로 리턴한다.

## 3. 소켓 인터페이스 함수

  소켓은 특수 파일중 하나이다. 그래서 소켓을 이용하여 네트워크 프로그래밍을 할 때는 소켓을 생성해 IP 주소와 연결한 후 서버와 클라이언트가 연결되면 소켓을 통해 읽고 쓰면 된다. </br>
  소켓을 이용해 데이터를 주고 받으려면 다양한 소켓 함수가 필요하며 이 함수들을 순서에 맞게 호출해야 한다.

  ### 소켓 인터페이스 함수의 종류
    socket(): 소켓 파일 기술자 생성
    bind(): 소켓 파일 기술자를 지정된 IP 주소/포트 번호와 결합
    listen(): 클라이언트의 연결 요청 대기
    connect(): 클라이언트가 서버에 접속 요청
    accept(): 클라이언트의 연결 요청 수락
    send(): 데이터 송신(SOCK_STREAM)
    recv(): 데이터 수신(SOCK_STREAM)
    sendto(): 데이터 송신(SOCK_DGRAM)
    recvfrom(): 데이터 수신(SOCK_DGRAM)
    close(): 소켓 파일 기술자 종료

    bind(), listen(), accept() 함수는 서버 측에서만 사용하고, connect() 함수는 클라이언트 측에서만 사용한다.
    나머지 socket(), recv(), send(), recvfrom(), sendto(), close() 함수는 서버와 클라이언트에서 모두 사용한다.

    socket(): 소켓 파일 기술자 생성
    --------------------------------------
    #include <sys/types.h>
    #include <sys/socket.h>

    int socket(int domain, int type, int protocol);

    * domain: 소켓 종류(유닉스 도메인 또는 인터넷 소켓)
    * type: 통신 방식(TCP 또는 UDP)
    * protocol: 소켓에 이용할 프로토콜
    --------------------------------------
    socket() 함수는 domain에 지정한 소켓의 형식과 type에 지정한 통신 방식을 지원하는 소켓을 생성함.
    protocol은 소켓에서 이용할 프로토콜로 보통은 0을 지정하고 시스템이 protocol 값을 결정함.
    domain에는 도메인 또는 주소 패밀리를 지정함. 유닉스 도메인 소켓을 생성할 경우 AF_UNIX를 지정하고 인터넷 소켓을 생성할 경우 AF_INET을 지정함.
    type에는 통신 방식에 따라 SOCK_STREAM이나 SOCK_DGRAM을 지정함.

    bind(): 소켓 파일 기술자를 지정된 IP 주소/포트 번호와 결합
    --------------------------------------
    #include <sys/types.h>
    #include <sys/socket.h>

    int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);

    * sockfd: socket() 함수가 생성한 소켓 기술자
    * addr: 소켓의 이름을 표현하는 구조체
    * addrlen: addr의 크기
    --------------------------------------
    socket() 함수로 생성한 소켓을 사용하려면 소켓을 특정 IP 및 포트 번호와 연결해야 한다.
    bind() 함수는 socket() 함수가 생성한 소켓 기술자 sockfd에 sockaddr 구조체인 addr에 지정한 정보를 연결하는데, 이 작업을 '소켓에 이름을 할당하기'라고 부른다.
    sockaddr 구조체에 지정하는 정보는 소켓의 종류, IP 주소, 포트 번호다. bind() 함수는 수행에 성공하면 0을, 실패하면 -1을 리턴한다.

    listen(): 클라이언트의 연결 요청 대기
    --------------------------------------
    #include <sys/types.h>
    #include <sys/socket.h>

    int listen(int sockfd, int backlog);

    * sockfd: socket 함수가 생성한 소켓 기술자
    * backlog: 최대 허용 클라이언트 수
    --------------------------------------
    소켓 sockfd에서 클라이언트의 연결을 받을 준비를 마쳤음을 알린다.
    접속이 가능한 클라이언트 수는 backlog에 지정함. listen() 함수는 소켓이 SOCK_STREAM 방식으로 통신할 때만 필요함.
    
