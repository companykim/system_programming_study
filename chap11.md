# chapter 11 시스템 V의 프로세스 간 통신

## 1. 시스템 V IPC의 공통 요소

  시스템 V IPC에서 공통적으로 사용하는 기본 요소는 키와 식별자다. 메시지 큐, 공유 메모리, 세마포어를 사용하려면 식별자가 있어야 한다.
  또한 시스템 V IPC에서는 현재 사용 중인 통신 방법의 상태를 확인하고 불필요한 것은 정리할 수 있는 관리 명령을 제공한다.

  ### 키와 식별자(ID)
    키의 생성
    키로 IPC_PRIVATE 지정: 식별자를 알아야 통신할 수 있으므로 IPC_PRIVATE를 키로 지정해 생성된 식별자를 서버와 클라이언트 모두 알 수 있게 해야 한다.
    fork() 함수로 생성된 부모-자식 프로세스 간 통신에서 유용하게 사용할 수 있다.
    ftok() 함수로 키 생성: ftok() 함수는 경로명과 숫자값을 받아서 키를 생성한다. 따라서 서버와 클라이언트가 같은 경로명과 숫자값을 지정하면 공통 식별자를 생성할 수 있다.

    같은 키로 생성된 식별자는 통신에 사용할 수 있다. 따라서 미리 정해진 키를 서버와 클라이언트 프로세스가 공유할 수 있게 해야 한다. 헤더 파일이나 환경 설정 파일에 키를 저장해 공유할 수 있다.

    ftok(): 키 생성하기
    ----------------------------------------
    #include <sys/types.h>
    #include <sys/ipc.h>

    key_t ftok(const char *pathname, int proj_id);

    * pathname: 파일 시스템에 이미 존재하는 임의의 파일 경로명
    * proj_id: 키값을 생성할 때 지정하는 임의의 번호(1~255)
    ----------------------------------------
    ftok() 함수는 pathname에 지정한 경로명과 proj_id에 지정한 정수값을 조합해 새로운 키를 생성한다. 
    경로명은 파일 시스템에 존재해야 하며 접근 권한이 있어야 한다. 이 키는 IPC 객체를 생성할 때 사용한다.
    키를 구성하는 32비트 중 처음 12비트에는 stat 구조체의 st_dev 값, 다음 12비트에는 st_ino 값이 저장된다.
    마지막 8비트에는 ftok() 함수의 두 번째 인자로 지정한 정수값이 저장된다.

    경로명으로 지정한 파일이 실제로 존재하고 있어서 stat() 함수를 실행할 수 있어야 한다. 따라서 키가 참조하고 있는 경로의 파일이 삭제되면 ftok() 함수는 오류를 발생시킨다.
    삭제된 파일을 다시 생성하고 같은 숫자를 id로 지정한 후 ftok() 함수로 키를 생성해도 다른 키가 생성된다.
    키의 마지막 8비트에는 정수값이 저장되므로 1~255 사이의 값을 지정해야 하며 이 값을 0으로 지정하면 안된다.
    key_t key;
    key = ftok("/home/keyfile", 1);

  ### IPC 공통 구조체
    struct ipc_perm {
      key_t            __key; 키값
      uid_t            uid; 구조체의 소유자 ID
      gid_t            gid; 구조체의 소유 그룹 ID
      uid_t            cuid; 구조체를 생성한 사용자 ID
      gid_t            cgid; 구조체를 생성한 그룹 ID
      unsigned short   mode; 구조체에 대한 접근 권한
      unsigned short   __seq; 일련 번호
    };

  ### 시스템 V IPC 정보 검색
    시스템 V IPC의 정보를 검색하고 현재 상태를 확인하는 명령은 ipcs다. ipcs 명령을 실행하는 동안에도 IPC의 상태가 변경될 수 있다.
    ipcs 명령은 검색하는 순간의 정확성만 보장하기 때문에 성태가 변경된 정보를 보려면 명령을 다시 수행해야 한다.

    기본형식
    ipcs [ihVmqsaclptu]

    일반 옵션
      -i id: id로 지정한 특정 요소에 대한 상세 정보를 출력한다. -m, -q, -s 중 하나와 결합해 사용함.
      -h: 도움말을 출력함
      -V: 버전 정보 출력

    자원 옵션
      -m: 공유 메모리 정보만 검색
      -q: 메시지 큐 정보만 검색
      -s: 세마포어 정보만 검색
      -a: 공유 메모리, 메시지 큐, 세마포어 모두의 정보만 검색. 이 옵션이 기본값

    출력 옵션: 출력 옵션은 하나만 지정
      -c: IPC 자원을 생성한 사용자와 소유자 정보를 출력
      -l: 사용할 수 있는 메시지 큐, 공유 메모리, 세마포어의 제한값을 출력함.
      -p: 자원의 생성자와 마지막 운영자의 PID를 촐력
      -t: 시간 정보를 출력
      -u: 요약 정보를 출력

    표현 형식: -l 옵션에만 적용
      -b: 크기 정보를 바이트 단위로 출력
      --human: 크기 정보를 사용자가 읽기 편한 형식으로 출력

    ipcs 명령의 사용
      ipcs 명령의 다양한 옵션을 적절히 조합해 원하는 정보를 검색할 수 있다.

  ### 시스템 V IPC 자원의 생성과 삭제
    시스템 V IPC 자원을 명령으로 생성할 때는 ipcmk 명령을 사용하고, 불필요한 IPC 자원을 삭제할 때는 ipcrm 명령을 사용함.

    ipcmk 명령
      ipcmk [options]

      -M size: size에 지정한 바이트 크기로 공유 메모리를 생성
      -Q: 메시지 큐를 생성
      -S number: number에 지정한 개수의 요소를 갖는 세마포어를 생성
      -p mode: 자원의 접근 권한을 지정함. 기본값은 0644

    ipcrm 명령: IPC 자원과 관련된 자료 구조를 모두 제거함. 제거 하기 위해서는 root 계정이거나 자원의 생성자 또는 사용자여야 한다.
      ipcrm [options]

      -a: 모든 자원을 제거함
      -M shmkey: shmkey로 생성한 공유 메모리의 마지막 연결이 해제된 후 공유 메모리를 제거한다.
      -m shmid: shmid로 지정한 공유 메모리를 삭제한다.
      -Q msgkey: msgkey로 생성한 메시지 큐를 제거한다.
      -q msgid: msgid로 지정한 메시지 큐를 삭제한다.
      -S semkey: semkey로 생성한 세마포어를 삭제한다.
      -s semid: semid로 지정한 semid로 지정한 세마포어를 삭제한다.

  ## 2. 메시지 큐

    메시지 큐는 파이프와 유사하다. 파이프는 스트림 기반으로 동작하고 메시지 큐는 메세지(또는 패킷) 단위로 동작한다. 각 메시지의 최대 크기는 제한되어 있다.
    우편함과 비슷하다고 생각하면 된다. 각 메시지에는 유형이 있어 수신 프로세스는 어떤 유형의 메시지를 받을 것인지 선택할 수 있다.
    메시지 큐 제어 함수는 메시지 큐를 제거하거나 상태 정보를 설정하고 읽어오는 등 메시지 큐에 대한 제어 기능을 수행함.

      msgget(): 메시지 큐 식별자 생성
      ----------------------------------------
      #include <sys/types.h>
      #include <sys/ipc.h>
      #include <sys/msg.h>

      int msgget(key_t key, int msgflg);

      * key: 메시지 큐를 구별하는 키
      * msgflg: 메시지 큐의 속성을 설정하는 플래그
      ----------------------------------------
      msgget() 함수는 인자로 키와 플래그를 받아 메시지 큐 식별자를 리턴한다.
      첫번째 인자인 key에는 IPC_PRIVATE나 ftok() 함수로 생성한 키를 지정함. 두번째 인자인 msgflg에는 플래그와 접근 권한을 지정한다. 
      IPC_CREAT: 새로운 키면 식별자를 새로 생성함.
      IPC_EXCL: 이미 존재하는 키면 오류가 발생함.

      메시지 큐 식별자와 관련된 메시지 큐와 IPC 구조체기 새로 생성되는 경우
      key가 IPC_PRIVATE다.
      key가 IPC_PRIVATE가 아니며 key에 지정한 식별자와 관련된 다른 메시지 큐가 없고 플래그(msgflg)에 IPC_CREAT가 설정되어 있다.
      이 두가지 경우가 아니면 msgget() 함수는 기존 메시지 큐의 식별자를 리턴한다.

      struct msqid_ds {
        struct ipc_perm msg_perm; IPC 공통 구조체(ipc_perm)이다.
        time_t    msg_stime; 마지막으로 메시지를 보낸 시각
        time_t    msg_rtime; 마지막으로 메시지를 읽은 시각
        time_t    msg_ctime; 마지막으로 메시지 큐의 권한을 바꾼 시각
        unsigned long    __msg_cbytes; 현재 메시지 큐에 있는 메시지의 총 바이트 수
        msgqnum_t    msg_qnum; 메시지 큐에 있는 메시지의 개수
        msglen_t    msg_qbytes; 메시지 큐의 최대 크기(바이트 수)
        pid_t    msg_lspid; 마지막으로 메시지를 보낸 프로세스의 PID
        pid_t    msg_lrpid; 마지막으로 메시지를 읽은 프로세스의 PID
      };

      식별자가 리턴할 때 메시지 큐 구조체의 값
      msg_perm.cuid, msg_perm.uid: 함수를 호출한 프로세스의 유효 사용자 ID로 설정
      msg_perm.cgid, msg_perm.gid: 함수를 호출한 프로세스의 유효 그룹 ID로 설정
      msg_perm.mode의 하위 9비트: msgflg 값의 하위 9비트로 설정
      msg_qnum, msg_lspid, msg_lrpid, msg_stime, msg_rtime: 0으로 설정
      msg_ctime: 현재 시간으로 설정
      msg_qbytes: 시스템의 제한값으로 설정

      예)
      key_t key;
      int id

      key = ftok("keyfile", 1);
      id = msgget(key, IPC_CREAT|0640);

      msgsnd(): 메시지 전송
      ----------------------------------------
      #include <sys/types.h>
      #include <sys/ipc.h>
      #include <sys/msg.h>

      int msgsnd(int msqid, const void *msgp, size_t msgsz, int msgflg);

      * msqid: msgget 함수로 생성한 메시지 큐 식별자
      * msgp: 메시지를 담고 있는 메시지 버퍼의 주소
      * msgsz: 메시지의 크기(0~시스템이 정한 최대값)
      * msgflg: 블록 모드(0)/비블록 모드(IPC_NOWAIT)
      ----------------------------------------
      msgsnd() 함수는 msgget() 함수가 리턴한 메시지 큐(msqid)를 통해 크기가 msgsz인 메시지를 메시지 버퍼(msgp)에 담아 전송함.
      msgflg에는 메시지 큐가 가득 찼을 때의 동작을 지정함. msgsz에는 전송하는 메시지의 크기를 지정함.
      msgflg에는 0이나 IPC_NOWAIT를 지정함. 
      
      메시지를 담고 있는 메시지 버퍼는 msgbuf 구조체를 사용함.
      struct msgbuf {
        long    mtype; 메시지 유형으로 양수를 지정함.
        char    mtext[1]; msgsnd() 함수의 msgsz로 지정한 크기의 버퍼로 메시지 내용이 저장된다.
      };

      msgrcv(): 메시지 수신
      ----------------------------------------
      #include <sys/types.h>
      #include <sys/ipc.h>
      #include <sys/msg.h>

      ssize_t msgrcv(int msgid, void *msgp, size_t msgsz, long msgtyp, int msgflg);

      * msqid: msgget() 함수로 생성한 메시지 큐 식별자
      * msgp: 메시지를 담고 있는 메시지 버퍼의 주소
      * msgsz: 메시지 버퍼의 크기
      * msgtyp: 읽어올 메시지의 유형
      * msgflg: 블록 모드(0)/비블록 모드(IPC_NOWAIT)
      ----------------------------------------
      msqid가 가리키는 메시지 큐에서 msgtyp이 지정하는 메시지를 읽어 msgp가 가리키는 메시지 버퍼에 저장한다.
      메시지 버퍼의 크기는 msgsz에 지정하고 msgflg는 메시지 큐가 비었을 때 어떻게 동작할 것인지 알려준다.
      msgtyp에 지정할 수 있는 값
      0: 메시지 큐의 가장 앞에 있는 메시지를 읽어온다.
      양수: 메시지 큐에서 msgtyp로 지정한 유형과 같은 메시지를 읽어온다.
      음수: 메시지 유형이 msgtyp로 지정한 값의 절대값과 같거나 작은 메시지를 읽어온다.

      msgflg에는 msgsnd() 함수처럼 블록 모드/비블록 모드를 지정함.
      MSG_COPY: 메시지 큐에서 메시지를 복사해오고 원본은 큐에 그대로 둔다. IPC_NOWAIT와 같이 사용함.
      MSG_EXCEPT: msgtyp의 값이 양수일 때 msgtyp에서 지정한 유형과 다른 유형의 메시지 중 첫번째 메시지를 가져온다.
      MSG_NOERROR: 메시지가 msgsz에 지정한 바이트보다 크면 메시지 내용을 잘라낸다.
      msgrcv() 함수의 수행이 성공하면 msqid_ds 구조체 항목에서 msg_qnum 값이 1 감소하고 msg_lrpid는 msgrcv() 함수를 호출한 프로세스의 ID로 설정됨.

      msgctl(): 메시지 제어
      ----------------------------------------
      #include <sys/types.h>
      #include <sys/ipc.h>
      #include <sys/msg.h>

      * msqid: msgget() 함수로 생성한 메시지 큐 식별자
      * cmd: 수행할 제어 기능
      * buf: 제어 기능에 사용되는 메시지 큐 구조체의 주소
      ----------------------------------------
      msqid로 지정한 메시지 큐에서 cmd에 지정한 제어를 수행함.
      buf는 cmd의 종류에 따라 제어값을 지정하거나 읽어오는데 사용함.
      cmd에 지정하는...
      IPC_STAT: 현재 메시지 큐의 정보를 buf로 지정한 메모리에 저장함.
      IPC_SET: 메시지 큐의 정보 중 msg_perm.uid, msg_perm.gid, msg_perm.mode, msg_qbytes 값을 세번째 인자로 지정한 값으로 바꾼다.
      IPC_RMID: msqid로 지정한 메시지 큐를 제거하고 관련된 데이터 구조체를 제거한다.
      IPC_INFO: 리눅스에서만 사용가능하며 메시지 큐의 제한값을 buf에 저장한다. buf는 msginfo 구조체로 형 변환해야 한다.

      struct msginfo {
        int msgpool; 메시지 데이터를 저장할 수 있는 버퍼 공간의 크기(KB)다.
        int msgmap; 메시지 맵의 최대 항목 수
        int msgmax; 한 메시지에 저장할 수 있는 메시지의 최대 크기(바이트)다.
        int msgmnb; 메시지 큐에 작성할 수 있는 최대 크기(바이트)로, msgget() 함수로 메시지 큐를 생성할 때 msg_qbytes 값 초기화에 사용한다.
        int msgmni; 메시지 큐의 최대 개수
        int msgssz; 메시지 세그먼트 크기
        int msgtql; 시스템에 있는 모든 메시지 큐의 최대 메시지 개수
        unsigned short msgseg; 세그먼트의 최대 개수
      };

  ## 3. 공유 메모리

    같은 메모리 공간을 2개 이상의 프로세스가 공유하는 것.
    같은 메모리 공간을 사용하므로 이를 통해 데이터를 주고 받을 수 있다.
    여러 프로세스가 메모리를 공유하고 있으므로 당연히 읽고 쓸 때 동기화가 필요하다.
    공유 메모리를 동기화하지 않으면 데이터가 손실될 수 있다.

  ### 공유 메모리 함수
    공유 메모리 제어 함수는 메시지 큐 제어 함수와 마찬가지로 공유 메모리 제거, 잠금 설정, 잠금 해제 등의 제어 기능을 수행한다.

    shmget(): 공유 메모리 식별자 생성
    ----------------------------------------
    #include <sys/ipc.h>
    #include <sys/shm.h>

    int shmget(key_t key, size_t size, int shmflg);

    * key: IPC_PRIVATE 또는 ftok() 함수로 생성한 키
    * size: 공유할 메모리의 크기
    * shmflg: 공유 메모리의 속성을 지정하는 플래그
    ----------------------------------------
    공유 메모리를 사용하려면 공유 메모리 식별자를 생성해야 한다.
    shmget() 함수는 인자로 키, 공유할 메모리의 크기, 플래그를 받아 공유 메모리 식별자를 리턴한다.
    
    공유 메모리 식별자와 관련된 공유 메모리와 데이터 구조체가 새로 생성되는 경우
    key가 IPC_PRIVATE다.
    key가 0이 아니며 다른 식별자와 관련되어 있지 않고, 플래그(msgflg)에 IPC_CREAT가 설정되어 있다.
    이 두 경우가 아니면 기존 공유 메모리의 식별자를 리턴함.

    struct shmid_ds {
      struct ipc_perm msg_perm; IPC 공통 구조체(ipc_perm)
      size_t    shm_segsz; 공유 메모리 세그먼트의 크기를 바이트 단위로 나타낸다.
      time_t    shm_atime; 마지막으로 공유 메모리를 연결(shmat)한 시간 
      time_t    shm_dtime; 마지막으로 공유 메모리의 연결을 해제(shmdt)한 시간
      time_t    shm_ctime; 마지막으로 공유 메모리의 접근 권한을 변경한 시간
      pid_tt_t    shm_lpid; 마지막으로 shmop 동작을 한 프로세스의 PID
      pid_t    shm_cpid; 공유 메모리를 생성한 프로세스의 PID
      shmatt_t    shm_nattch; 공유 메모리를 연결하고 있는 프로세스의 개수
      ...
    };

    shm_perm.cuid, shm_perm.uid: 함수를 호출한 프로세스의 유효 사용자 ID로 설정
    shm_perm.cgid, shm_perm.gid: 함수를 호출한 프로세스의 유효 그룹 ID로 설정
    shm_perm.mode의 하위 9비트: msgflg 값의 하위 9비트로 설정
    shm_segsz: size 값으로 설정
    shm_lpid, shm_nattch, shm_atime, shm_dtime: 0으로 설정
    shm_ctime: 현재 시각으로 설정

    shmat(): 공유 메모리 연결
    ----------------------------------------
    #include <sys/types.h>
    #include <sys/shm.h>

    void *shmat(int shmid, const void *shmaddr, int shmflg);

    * shmid: shmget() 함수로 생성한 공유 메모리 식별자
    * shmaddr: 공유 메모리를 연결할 주소
    * shmflg: 공유 메모리에 대한 읽기/쓰기 권한
    ----------------------------------------
    생성된 공유 메모리를 사용하려면 공유 메모리를 프로세스의 데이터 영역과 연결해야 한다.
    shmget() 함수로 생성한 공유 메모리의 식별자를 shmid에 지정하고, shmaddr에 공유 메모리를 연결할 주소를 지정함.
    값이 NULL이면 시스템이 알아서 적절한 주소에 공유 메모리를 연결한다.
    shmflg는 플래그로, 0이면 공유 메모리에 대해 읽기와 쓰기가 가능하고, SHM_RDONLY면 읽기만 가능.

    shmdt(): 공유 메모리 연결 해제
    ----------------------------------------
    #include <sys/types.h>
    #include <sys/shm.h>

    int shmdt(const void *shmaddr);

    * shmaddr: 연결을 해제할 공유 메모리의 시작 주소
    ----------------------------------------
    shmaddr에 공유 메모리의 시작 주소를 지정해야 한다.
    공유 메모리의 사용이 끝나면 shmdt() 함수를 사용해 공유 메모리의 연결을 해제해야 한다.

    shmctl(): 공유 메모리 제어
    ----------------------------------------
    #include <sys/ipc.h>
    #include <sys/shm.h>

    int shmctl(int shmid, int cmd, struct shmid_ds *buf);

    * shmid: shmget() 함수로 생성한 공유 메모리 식별자
    * cmd: 수행할 제어 기능
    * buf: 제어 기능에 사용되는 공유 메모리 구조체의 주소
    ----------------------------------------
    shmid가 가리키는 공유 메모리에 cmd로 지정한 제어 기능을 수행한다.
    buf에는 제어 기능에 따라 사용되는 공유 메모리 구조체의 주소를 지정함.
    cmd에 지정할 수 있는 값
    IPC_STAT: 현재 공유 메모리의 정보를 buf로 지정한 메모리에 저장
    IPC_SET: 공유 메모리의 정보 중 shm_perm.uid, shm_perm.gid, shm_perm.mode 값을 세 번째 인자로 지정한 값으로 바꾼다. 이 명령은 root 권한이 있거나 유효 사용자 ID인 경우만 사용할 수 있다.
    IPC_RMID: shmid로 지정한 공유 메모리를 제거하고 관련된 데이터 구조체를 제거한다.
    IPC_INFO: 리눅스에서만 사용할 수 있으며, 공유 메모리의 시스템 제한값을 buf에 저장한다.
    struct shminfo {
      unsigned long shmmax; 세그먼트의 최대 크기
      unsigned long shmmin; 세그먼트의 최소 크기로, 항상 1이다.
      unsigned long shmmni; 세그먼트의 최대 개수
      unsigned long shmseg; 한 프로세스에 연결될 수 있는 세그먼트의 최대 개수
      unsigned long shmall; 시스템 전체에서 공유 메모리로 사용할 수 있는 페이지의 최대 개수
    };
    SHM_INFO: 리눅스에서만 사용가능하며, 공유 메모리가 사용하는 자원의 정보를 shm_info 구조체에 저장함.
    struct shm_info {
      int used_ids; 현재 생성된 세그먼트의 개수
      unsigned long shm_tot; 공유 메모리 페이지의 전체 개수
      unsigned long shm_rss; 사용 중인 공유 메모리 페이지의 개수
      unsigned long shm_swp; 스왑된 공유 메모리 페이지의 개수
      unsigned long swap_attempts;
      unsigned long swap_successes;
    };
    SHM_LOCK: 공유 메모리 세그먼트를 잠금.
    SHM_UNLOCK: 공유 메모리 세그먼트의 잠금을 해제함.

## 4. 세마포어

  프로세스 사이의 동기를 맞추는 기능을 제공한다.
  프로세스들이 공유 영역에 접근하는 순서를 정하는 방법 중 하나이다.

  ### 기본 개념
    세마포어는 한번에 한 프로세스만 작업을 수행하는 부분에 접근해 잠그거나 다시 잠금을 해제하는 기능을 제공하는 정수형 변수
    이 정수형 변수는 함수를 통해 값을 변경한다.
    잠금함수는 p(), 잠금 해제 함수는 v()

    ### 세마포어의 기본 동작 구조
    세마포어는 중요한 처리 부분에 들어가기 전에 p() 함수를 실행해 잠금 기능을 수행하고, 처리를 마치면 다시 v() 함수를 실행해 잠금을 해제한다.
    잠금 기능을 수행 중인 동안에는 다른 프로세스가 처리 부분의 코드를 실행할 수 없다. sem은 세마포어 값을 의미한다.

    p(sem);  /* 잠금 */
    중요한 처리 부분(Critical section)
    v(sem);  /* 잠금 해제 */

    p() 함수의 기본 동작 구조
    p(sem) {
          while sem == 0 do wait;
          sem 값을 1 감소;
    }
    초기 sem값은 1이다. p() 함수는 sem이 아니면 다른 프로세스가 처리 부분을 수행하고 있다는 의미이므로 값이 1이 될때까지 기다려야 한다.

    v() 함수의 기본 동작 구조
    v(sem) {
          sem 값을 1 증가;
          if (대기 중인 프로세스가 있으면)
                대기 중인 첫 번째 프로세스를 동작시킨다.
    }

  ### 세마포어 함수

    semget(): 세마포어 생성
    ----------------------------------------
    #include <sys/types.h>
    #include <sys/ipc.h>
    #include <sys/sem.h>

    int semget(key_t key, int nsems, int semflg);

    * key: IPC_PRIVATE 또는 ftok() 함수로 생성한 키
    * nsems: 생성할 세마포어 개수
    * semflg: 세마포어 접근 속성
    ----------------------------------------
    semget() 함수는 이자로 키와 생성할 세마포어 개수, 플래그를 받고 세마포어 식별자를 리턴한다.
    세마포어는 집합 단위로 처리되므로 한 식별자에 여러 세마포어가 생성된다.
    첫번째 인자인 key에는 IPC_PRIVATE나 ftok() 함수로 생성한 키를 지정한다. 두 번째 인자인 nsems에는 생성할 세마포어 개수를 지정한다.
    세번째 인자인 semflg에는 플래그와 접근 권한을 지정한다. 플래그는 msgget() 함수와 마찬가지로 IPC_CREAT나 IPC_EXCL을 사용함.

    세마포어 식별자와 관련된 세마포어와 데이터 구조체가 새로 생성되는 경우
    1. key가 IPC_PRIVATE다.
    2. key와 관련된 다른 세마포어 집합이없고 플래그(semflg)에 IPC_CREAT가 설정되어 있다.

    struct semid_ds {
      struct ipc_perm sem_perm; IPC 공통 구조체
      ushort_t    sem_nsems; 세마포어 연산을 수행한 마지막 시간
      time_t      sem_ctime; 세마포어의 접근 권한을 변경한 마지막 시간
      time_t      sem_otime; 세마포어 집합의 세마포어 개수
    };

    새로운 세마포어 식별자를 리턴할 때 구조체
    sem_perm.cuid, sem_perm.uid: 함수를 호출한 프로세스의 유효 사용자 ID로 설정됨.
    sem_perm.cgid, sem_perm.gid: 함수를 호출한 프로세스의 유효 그룹 ID로 설정됨.
    sem_perm.mode: semflg 값으로 설정됨.
    sem_nsems: nsems 값으로 설정됨. 세마포어 집합이 생성되지 않으면 0으로 설정될 수도 있다.
    sem_otime: 0으로 설정됨.
    sem_ctime: 현재 시각으로 설정됨.

    semctl(): 세마포어 제어
    ----------------------------------------
    #include <sys/types.h>
    #include <sys/ipc.h>
    #include <sys/sem.h>

    int semctl(int semid, int semnum, int cmd, ...);

    * semid: semget() 함수로 생성한 세마포어 식별자
    * semnum: 기능을 제어할 세마포어 번호
    * cmd: 수행할 제어 명령
    * ...: 제어 명령에 따라 필요시 사용할 세마포어 공용체의 주소 (선택 사항)
    ----------------------------------------
    semid로 식별되는 세마포어 집합에서 semnum으로 지정한 세마포어에 cmd로 지정한 제어 기능을 수행한다.
    
    cmd에 지정할 수 있는 값
    IPC_STAT: 현재 세마포어의 정보를 arg.buf로 지정한 메모리에 저장한다.
    IPC_SET: 세마포어 정보 중 sem_perm.uid, sem_perm.gid, sem_perm.mode 값을 네번째 인자로 지정한 값으로 변경한다. 이 명령은 root 권한이 있거나 유효 사용자 ID일 경우만 사용할 수 있다.
    IPC_RMID: semid로 지정한 세마포어와 관련된 데이터 구조체를 제거한다.
    IPC_INFO: 리눅스에서만 사용할 수 있으며 공유 메모리의 시스템 제한값을 buf에 저장한다. 이때 buf는 seminfo 구조체로 형 변환을 해야 한다.

    struct seminfo {
      int semmap; 세마포어 맵에 있는 항목의 개수
      int semmni; 세마포어 집합의 최대 개수
      int semmns; 전체 세마포어 집합에서 세마포어의 최대 개수
      int semmnu; 시스템 전체에서 undo 구조체의 최대 개수
      int semmsl; 한 집합에서 세마포어의 최대 개수
      int semopm; semop()에 가능한 최대 연산 개수
      int semume; 프로세스 당 가능한 undo 항목의 최대 개수
      int semusz; sem_undo 구조체의 크기
      int semvmx; 세마포어의 최대값
      int semaem; 세마포어 재정리(SEM_UNDO)를 위해 기록할 수 있는 최대값
    };

    GETALL: 세마포어의 semncnt 값을 읽어온다.
    GETNCNT: 세마포어의 semval 값을 읽어온다.
    GETVAL: 세마포어의 semval 값을 arg.val로 설정한다.
    GETPID: 세마포어의 sempid 값을 읽어온다.
    GETZCNT: 세마포어의 semzcnt 값을 읽어온다.
    SETALL: 세마포어 집합에 있는 모든 세마포어의 semval 값을 arg.array가 가리키는 배열값으로 설정한다.

    semctl() 함수의 리턴값은 cmd에 따라 달라진다. cmd가 GETVAL이면 semval 값을, GETPID면 sempid 값을 리턴한다.
    cmd가 GETNCNT면 semncnt 값을, GETZCNT면 semzcnt 값을 리턴한다.

    union semun {
      int    val;
      struct semid_ds *buf;
      unsigned short *array;
      struct seminfo *__buf;  /* 리눅스에서만 사용 */
    };

    semop(): 세마포어 연산
    ----------------------------------------
    #include <sys/types.h>
    #include <sys/ipc.h>
    #include <sys/sem.h>

    int semop(int semid, struct sembuf *sops, size_t nsops);

    * sedmid: semget() 함수로 생성한 세마포어 식별자
    * sops: sembuf 구조체의 주소
    * nsops: sops가 가리키는 구조체의 크기
    ----------------------------------------
    semid가 가리키는 세마포어에 크기가 nsops인 sembuf 구조체로 지정한 연산을 실행한다.

    struct sembuf {
      unsigned short  sem_num; 세마포어 번호
      short           sem_op; 세마포어 연산
      short           sem_flg; 연산을 위한 플래그, IPC_NOWAIT나 SEM_UNDO를 지정
                              SEM_UNDO: 프로세스가 비정상적으로 갑자기 종료될 때 세마포어 동작을 취소시킨다.
    };
    sem_op 항목은 semop 함수가 수행할 기능을 정수로 나타낸다.
