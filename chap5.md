# chapter 5 시스템 정보

  ## 1. 시스템 정보 검색
  
  리눅스 시스템 정보에는 시스템에 설치된 운영체제에 관한 정보, 호스트명 정보, 하드웨어 종류에 관한 정보가 있다. </br> 
  하드웨어에 따라 사용할 수 있는 자원의 최대값이 설정되어 있는데, 함수를 사용해 자원값을 검색하거나 수정할 수 있다. </br>
  자원값에는 최대 프로세스 개수, 프로세스 하나가 열 수 있는 최대 파일 개수, 메모리 페이지 크기 등이 있다. </br>
  시스템의 기본 환경을 검색하는데 사용하는 다양한 구조체와 매개변수를 제공한다. </br>

  ### 운영체제 기본 정보 검색
    시스템에 설치된 운영체제의 이름과 버전, 호스트명, 하드웨어 종류 등을 검색하려면 uname 명령을 사용한다.
    uname 명령에 -a 옵션을 지정하면 현재 시스템에 설치되어 있는 운영체제 정보가 출력된다.

    Linux / dbos1 / 4.18.0-305.el8.x86_64 / #1 SMP Thu Apr 29 08:54:30 EDT 2021 / x86_64 / x86_64 / x86_64 / GNU/Linux
    커널명/호스트명/커널 릴리즈/커널 버전/하드웨어명/프로세서명/플랫폼명/운영체제명

    uname: 운영체제 정보 검색                                 
    -------------------------------------------
    #include <sys/utsname.h>

    int uname(struct utsname *buf);

    * buf: utsname 구조체 주소
    -------------------------------------------
    uname() 함수는 운영체제 정보를 검색해 utsname 구조체에 저장.

    struct ustname {
      struct sysname[]; 현재 운영체제의 이름을 저장한다.
      struct nodename[]; 네트워크를 통해 통신할 때 사용하는 시스템의 이름을 저장한다.
      struct release[]; 운영체제의 릴리즈 번호를 저장한다.
      struct version[]; 운영체제의 버전 번호를 저장한다.
      struct machine[]; 운영체제가 동작하는 하드웨어의 표준 이름(아키텍쳐)을 저장한다.
    };
    
  ### 시스템 자원 정보 검색
    리눅스 시스템에서는 하드웨어에 따라 사용할 수 있는 자원의 최대값을 설정해놓았다.

    시스템 자원 정보 검색: sysconf()
    -------------------------------------------
    #include <unistd.h>

    long sysconf(int name);

    * name: 검색할 정보를 나타내는 상수
    -------------------------------------------
    검색하려는 시스템 정보를 나타내는 상수를 인자로 받고 현재 설정되어 있는 시스템 자원값 또는 옵션값을 리턴한다.

    POSIX.1에서 정의한 주요 상수
    | 상수 | 설명 |
    | _SC_ARG_MAX | exec() 계열 함수에 사용하는 인자의 최대 크기 |
    | _SC_CHILD_MAX | 한 UID에 허용되는 최대 프로세스 개수 |
    | _SC_HOST_NAME_MAX | 호스트명의 최대 길이 |
    | _SC_LOGIN_NAME_MAX | 로그인명의 최대 길이 |
    | _SC_CLK_TCK | 초당 클록 틱 수 |
    | _SC_OPEN_MAX | 프로세스당 열 수 있는 최대 파일 수 |
    | _SC_PAGE_SIZE | 시스템 메모리의 페이지 크기 |
    | _SC_VERSION | 시스템이 지원하는 POSIX.1의 버전 |

    파일과 디렉토리 자원 검색: fpathconf() / pathconf()
    -------------------------------------------
    #include <unistd.h>

    long fpathconf(int fd, int name);
    long pathconf(const char *path, int name);

    * fd: 파일 기술자
    * path: 파일이나 디렉토리 경로
    * name: 검색할 정보를 지정하는 상수
    -------------------------------------------
    fpathconf()는 열린 파일의 파일 기술자인 fd를 인자로 받아 이 파일과 관련된 자원값이나 옵션값 정보를 검색한다.
    pathconf()는 path에 지정한 파일이나 디렉토리와 관련해 설정된 자원값이나 옵션값을 리턴한다. name에는 검색할 정보를 나타내는 상수를 지정한다.
    
    검색에 사용할 수 있는 주요 상수
    _PC_LINK_MAX: 파일에 가능한 최대 링크 수
    _PC_NAME_MAX: 파일명의 최대 길이를 바이트 크기로 표시
    _PC_PATH_MAX: 상대 경로명의 최대 길이를 바이트 크기로 표시

  ## 2. 사용자 정보 검색

  리눅스 시스템에서 알 수 있는 사용자 정보에는 각 사용자에 관한 정보, 그룹에 관한 정보, 로그인 기록 정보가 있다. </br>
  이 정보들과 직접적으로 관련이 있는 파일은 패스워드 파일(/etc/passwd)과 섀도우 파일(/etc/shadow), 그룹 파일(/etc/group)이다.

  ### 로그인명과 UID 검색
    사용자 정보 중 가장 기본적인 정보는 로그인명이다. 사용자 계정을 등록할 때 로그인명과 사용자ID가 지정된다.

    getlogin(): 로그인명 검색
    -------------------------------------------
    #include <unistd.h>

    char *getlogin(void);
    -------------------------------------------
    /var/run/utmp 파일을 검색해 현재 프로세스를 실행한 사용자 로그인명을 찾아 리턴한다. (void는 인자로 아무것도 지정하지 않음을 의미함.)
    만약 이 프로세스를 실행한 사용자가 로그아웃했거나 rsh 등으로 원격에서 실행한 프로세스에서 getlogin() 함수를 호출하면 사용자명을 찾지 못하고 널 포인터를 리턴하므로 주의해야 한다.

    getuid()/geteuid(): UID 검색
    -------------------------------------------
    #include <unistd.h>
    #include <sys/types.h>

    uid_t getuid(void);
    uid_t geteuid(void);
    -------------------------------------------
    getuid()는 현재 프로세스의 실제 사용자 ID를, geteuid()는 유효 사용자 ID를 리턴한다.

    리눅스 시스템에서 프로세스와 관련된 UID
    실제 사용자 ID(RUID): 로그인할 때 사용한 로그인명에 대응하는 UID로, 프로그램을 실행하는 사용자를 나타낸다.
    유효 사용자 ID(EUID): 프로세스에 대한 접근 권한을 부여할 때 사용한다.

  ### 패스워드 파일 검색
    /etc/passwd 파일에는 로그인명, UID, GID, 사용자의 홈 디렉토리, 로그인 셸 등 사용자에 관한 기본적인 정보가 들어있다.

    /etc/passwd 파일의 구조
    이 파일은 사용자 정보를 각 행에 저장하며, 각 행에는 콜론(:)으로 구분된 정보가 있다.

    user1 : x : 1001 : 1001 : sample user : /home/user1 : /bin/bash
    로그인명 패스워드 UID GID 계정설명 홈 디렉토리 로그인셸

    리눅스에서는 사용자 계정을 등록할 때 UID와 같은 값으로 GID를 생성한다.

    passwd 구조체
    /etc/passwd 파일의 정보를 읽어오려면 passwd 구조체를 사용해야 한다.

    struct passwd {
      char *pw_name; 로그인명을 저장
      char *pw_passwd; 암호를 저장
      char pw_uid; uid를 저장
      char pw_gid; gid를 저장
      char *pw_gecos; 사용자 실명이나 기타 정보를 저장
      char *pw_dir; 홈 디렉토리를 저장
      char *pw_shell; 로그인 셸을 저장
    }

    getpwuid(): UID로 /etc/passwd 파일 읽기
    -------------------------------------------
    #include <sys/types.h>
    #include <pwd.h>

    struct passwd *getpwuid(uid_t uid);

    * uid: 검색할 UID
    -------------------------------------------
    /etc/passwd 파일에서 uid를 찾아 passwd 구조체에 결과를 저장하고 주소를 리턴한다.

    getpwnam(): 이름으로 passwd 파일 읽기
    -------------------------------------------
    #include <sys/types.h>
    #include <pwd.h>

    struct passwd *getpwnam(const char *name);

    * name: 로그인명
    -------------------------------------------
    로그인명을 받아 /etc/passwd 파일에서 사용자 정보를 검색한 후 검색 결과를 passwd 구조체에 저장하고 주소를 리턴한다.

    /etc/passwd 파일을 순차적으로 읽기: getpwent(), setpwent(), endpwent(), fgetpwent()
    -------------------------------------------
    #include <sys/types.h>
    #include <pwd.h>

    struct passwd *getpwent(void);
    void setpwent(void);
    void endpwent(void);
    struct passwd *fgetpwent(FILE *stream);

    * stream: 파일 포인터
    -------------------------------------------
    /etc/passwd 파일에서 사용자 정보를 순차적으로 읽어온다.
    setpwent()는 /etc/passwd 파일의 오프셋을 파일의 처음에 놓는다.
    endpwent()는 /etc/passwd 파일을 닫는다.

  ### 섀도우 파일 검색

    /etc/passwd 파일은 누구나 읽을 수 있기에 보안 문제가 발생할 수 있기 때문에 현재의 리눅스/유닉스 시스템은 사용자 패스워드를 /etc/shadow 파일에 별도로 저장한다.

    /etc/shadow 파일의 구조: 사용자의 패스워드를 각행에 저장한다.

    user1 : $6$9sC(생략). : 18705 : 0 : 99999 warn(:7:) inactive( :) expire( :) flag( ) 
    로그인ID : 패스워드 : 최종변경일 : min : max

    spwd 구조체
    /etc/shadow 파일은 사용자 패스워드 정보와 패스워드의 주기 정보를 저장한다.

    struct spwd {
      char *sp_namp; 로그인명을 저장
      char *sp_pwdp; 사용자 계정의 패스워드를 암호화해 저장
      int sp_lstchg; 패스워드를 변경한 날짜 정보
      int sp_min; 변경된 패스워드를 사용해야 하는 최소 일 수
      int sp_max; 현재 패스워드를 사용할 수 있는 최대 일 수
      int sp_warn; 패스워드를 변경할 날이 되기 전에 경고를 시작하는 일 수
      int sp_inact; 패스워드가 만료된 이후 사용자 계정이 정지될 때까지의 일 수
      int sp_expire; 사용자 계정이 만료되는 날짜 정보로, 1970년 1월 1일부터 일 수로 표시한다.
      unsigned int sp_flag; 나중에 사용하기 위해 예약된 공간으로, 현재는 사용하지 않음.
    };

    getspnam(): /etc/shadow 파일 검색
    -------------------------------------------
    #include <shadow.h>

    struct spwd *getspnam(const char *name);

    * name: 검색할 사용자명
    -------------------------------------------
    인자로 지정한 사용자의 패스워드 정보를 읽어온다.

    /etc/shadow 파일을 순차적으로 읽기ㅖ: getspent(), setspent(), endspent(), fgetspent()
    -------------------------------------------
    *include <shadow.h>

    struct spwd *getspent(void);
    void setspent(void);
    void endspent(void);
    struct spwd *fgetspent(FILE *stream);

    * stream: 파일 포인터
    -------------------------------------------
    /etc/shadow 파일에서 패스워드 정보를 순차적으로 읽어오며, / etc/shadow 파일의 끝을 만나면 널 포인터를 리턴한다.
    setspent()는 /etc/shadow 파일의 오프셋을 파일의 처음으로 위치시키고, endspent()는 /etc/shadow 파일을 닫는다.
    fgetspent()는 /etc/shadow 파일이 아닌 파일 포인터로 지정한 다른 파일에서 패스워드 정보를 읽어온다. /etc/shadow 파일과 동일해야 함.

  ### 그룹 정보 검색
    리눅스 시스템에서 사용자는 하나 이상의 그룹에 속한다.

    getgid(), getegid(): 그룹 ID 검색하기
    -------------------------------------------
    #include <unistd.h>
    #include <sys/types.h>

    gid_t getgid(void);
    gid_t getegid(void);
    -------------------------------------------
    getgid() 함수는 실제 그룹 ID를, getegid()는 유효 그룹 ID를 리턴한다.
    실제 그룹 ID(RGID)는 로그인할 때 사용한 사용자 계정의 기본 그룹이다. 유효 그룹 ID는 프로세스 접근 권한을 부여할 때 사용함.

  ### 그룹 정보 파일 검색
    리눅스에서는 그룹 정보를 /etc/group 파일에 별도로 저장함. 사용자가 속한 그룹 중 /etc/passwd 파일의 GID 항목에 지정된 그룹이 기본 그룹이며, 2차 그룹은 /etc/group 파일에서 지정함.

    /etc/group 파일의 구조
    /etc/group 파일에 한 행씩 저장되며 항목은 콜론(:)으로 구분됨.

    group 구조체
    struct group {
      char *gr_name; 그룹명을 저장
      char *gr_passwd; 그룹 패스워드를 저장. 보통은 공백이다. 
                       그룹 패스워드를 지정하면 사용자 패스워드처럼 암호화된 문자가 저장됨.
                       그룹 패스워드가 지정되어 있으면 사용자는 newgrp 명령을 사용해 다른 그룹으로 변경할 때 이 패스워드를 입력해야 한다.
      gid_t gr_gid; 그룹 ID 번호를 저장
      char **gr_mem; 그룹의 멤버인 로그인명을 저장
    };

    getgrnam(), getgrgid(): /etc/group 파일 검색
    -------------------------------------------
    #include <sys/types.h>
    #include <grp.h>

    struct group *getgrnam(const char *name);
    strurct group *getgrgid(gid_t gid);

    * name: 검색하려는 그룹명
    * gid: 검색하려는 그룹의 ID
    -------------------------------------------
    getgrnam()은 검색하려는 그룹명을 읽어오고, getgrgid()는 검색하려는 그룹의 ID를 읽어옴.

    getgrent(), setgrent(), endgrent(), fgetgrent(): /etc/group 파일을 순차적으로 읽기
    -------------------------------------------
    #include <sys/types.h>
    #include <grp.h>

    struct group *getgrent(void);
    void setgrent(void);
    void endgrent(void);
    struct group *fgetgrent(FILE *stream);

    * stream: 파일 포인터
    -------------------------------------------
    /etc/group 파일에서 그룹 정보를 순차적으로 읽어오며 /etc/group 파일의 끝을 만나면 널 포인터를 리턴한다.
    setgrent()는 /etc/group 파일의 오프셋을 파일의 처음으로 위치시킨다. endgrent()는 /etc/group 파일을 닫는다.
    fgetgrent()는 /etc/group 파일이 아닌 파일 포인터로 지정한 다른 파일에서 그룹 정보를 읽어온다. 이 파일의 구조는 /etc/group 파일과 동일해야 한다.

  ### 로그인 검색
    who 명령으로 현재 시스템에 로그인한 사용자 정보를 검색할 수 있다. 
    last 명령으로는 시스템의 부팅 시각 정보나 사용자 로그인 기록 등을 검색할 수 있다. 우분투 리눅스에서는 이 정보들을 /var/run/utmp와 /var/log/wtmp 파일에 저장한다.
    이 두 파일은 바이너리 파일이므로 vi로는 편집이 불가능하다. 이 파일에서 정보를 읽어오려면 파일의 구조와 관련된 구조체와 함수가 필요하다. 시스템에 따라 /utmp/wtmp를 사용하거나 /utmpx/wtmpx를 사용할 수 있다.

    utmp 구조체
    utmp 파일과 wtmp 파일은 구조가 같다.
    sturct utmp {
      short ut_type; 현재 읽어온 항목에 저장된 데이터 형식을 나타낸다.
      pid_t ut_pid; 로그인 프로세스의 PID
      char ut_line[UT_LINESIZE]; 사용자가 로그인한 장치명을 나타낸다.
      char ut_id[4]; 터미널 이름이거나 /etc/inittab 파일에서 읽어온 ID
      char ut_user[UT_NAMESIZE]; 사용자명을 저장
      char ut_host[UT_HOSTSIZE]; 
      sturct exit_status ut_exit; 프로세스가 DEAD_PROCESS인 경우 프로세스의 종료 상태 저장
      long ut_session; 해당 정보의 세션 번호
      struct timeval ut_tv; 해당 정보를 마지막으로 변경한 시각
      int32_t ut_addr_v6[4]; 원격 접속한 경우 원격 호스트의 인터넷 주소를 저장함.
      char __unused[20]; 추후 사용을 위해 예약해둔 부분
    };

    ut_type에 사용할 수 있는 상수

    | 이름 | 상수 | 의미 |
    | EMPTY | 0 | 빈 항목 |
    | RUN_LVL | 1 | 시스템의 런레벨이 변경되었음을 나타냄 |
    | BOOT_TIME | 2 | 시스템 부팅 시각 |
    | NEW_TIME | 3 | 시스템 클럭이 변경된 다음의 시간 정보 |
    | OLD_TIME | 4 | 시스템 클럭이 변경되기 전의 시간 정보 |
    | INIT_PROCESS | 5 | init이 생성한 프로세스임을 나타냄 |
    | LOGIN_PROCESS | 6 | 사용자 로그인을 위한 세션 리더 프로세스 |
    | USER_PROCESS | 7 | 일반 프로세스 |
    | DEAD_PROCESS | 8 | 종료한 프로세스 |
    | ACCOUNTING | 9 | 사용하지 않는 항목 |

    getutent(), setutent(), endutent(): /var/log/utmp 파일 순차적으로 읽기
    -------------------------------------------
    #include <utmp.h>

    struct utmp *getutent(void);
    void setutent(void);
    void endutent(void);
    int utmpname(const char *file);

    * fime: 지정할 파일명
    -------------------------------------------
    getutent()는 /var/run/utmp 파일에서 로그인 정보를 순차적으로 읽어오며, /var/run/utmp 파일의 끝을 만나면 널 포인터를 리턴한다.
    setutent()는 /var/run/utmp 파일의 오프셋을 파일의 시작에 위치시키며, endutent()는 /var/run/utmp 파일을 닫는다.
    utmpname()은 로그인 정보 파일을 file로 지정한 다른 파일로 변경한다.

  ## 3. 시간 관리 함수

    리눅스 시스템은 1970년 1월 1일 0시 0초부터 현재까지 경과한 시간을 초 단위로 저장하고 이를 기준으로 시간 정보를 관리한다.

    ### 기본 시간 정보 확인

    time(): 초 단위로 현재 시간 정보 얻기
    -------------------------------------------
    #include <time.h>

    time_t time(time_t *tloc);

    * tloc: 검색할 시간 정보를 저장할 주소
    -------------------------------------------
    1970년 1월 1일 0시 0초부터 현재까지 경과한 시간을 초 단위로 알려준다.

    gettimeofday(): 마이크로초 단위로 시간 정보 얻기
    -------------------------------------------
    #include <sys/time.h>

    int gettimeofday(struct timeval *tv, struct timezone *tz);
    int settimeofday(const struct timeval *tv, const struct timezone *tz);

    * tv: 시간 정보 구조체 주소
    * tz: 시간대 정보 구조체 주소
    -------------------------------------------
    시간 정보를 timeval 구조체에 저장해 리턴함. tz는 시간대를 나타내는 구조체이지만 사용하지 않기에 NULL로 지정해도 됨.
    tv가 null이면 시간정보를 읽어올 수 없다.
    
    struct timeval {
      time_t tv_sec; 초
      time_t tv_usec; 마이크로초
    }

    ### 시간대 정보
      리눅스는 기본 시간대 정보를 환경 변수 TZ에 설정함.
      시스템의 기본 시간대 정보는 /etc/default/init 파일에 'TZ_ROK'와 같은 형태로 지정되어 있음. 현재 지역의 시간대를 설정하는 함수를 제공함.

      tzset(): 시간대 설정
      -------------------------------------------
      #include <time.h>

      void tzset(void);
      -------------------------------------------
      현재 지역의 시간대로 설정함. 이 함수를 호출하면 전역 변수에 정보가 설정된다.

      extern char *tzname[2]; 지역 시간대와 보정된 시간대명을 약어로 저장한다.
      extern long timezone; UTC와 지역 시간대와 시차를 초 단위로 저장한다.
      extern int daylight; 서머타임제를 시행하면 0이 아니고, 그렇지 않으면 0이다.

    ### 시간의 형태 변환
      초 단위 시간을 사람이 이해할 수 있는 형태로 변환해서 출력해야 한다.

      시간 정보 분해: tm 구조체
      시간을 연, 월, 일, 분, 시, 초로 나누어 저장하므로 사용자가 원하는 형태로 출력할 수 있다.
      struct tm {
        int tm_sec; 초
        int tm_min; 분
        int tm_hour; 시
        int tm_mday; 일
        int tm_mon; 월, 0은 1월, 11은 12월
        int tm_year; 연도, 연도-1900 값을 리턴함. 리턴값에 1900을 더해야 실제 연도를 출력할 수 있다.
        int tm_wday; 요일, 0은 일, 6은 토요일
        int tm_yday; 일수
        int tm_isdst; 서머타임제 시행 여부. 1이면 실시중임을 의미함.
      };

      gmtime(), localtime(): 초 단위 시간 정보 분해
      -------------------------------------------
      #include <time.h>

      struct tm *gmtime(const time_t *timep);
      struct tm *localtime(const time_t *timep);

      * time: 초 단위 시간 정보를 저장한 주소
      -------------------------------------------
      초 단위 시간 정보를 인자로 받아 tm 구조체 형태로 리턴함.
      gmtime()은 UTC 시간대를 기준으로 시간을 분해한다. localtime()은 UTC 대신 지역 시간대를 기준으로 시간을 처리한다.

      mktime(): 초 단위 시간으로 역산
      -------------------------------------------
      #include <time.h>

      time_t mktime(struct tm *tm);

      * tm: 시간 정보를 저장한 tm 구조체 주소
      -------------------------------------------
      mktime()은 gmtime()과 localtime()과 반대 역할을 수행함. tm 구조체 형태의 시간을 인자로 받아 이를 1970년 1월 1일 0:0:0부터 얼마나 지났는지 초단위로 계산해 리턴함.

    ### 형식 지정 시간 출력
      ctime(): 초 단위 시간을 변환해 출력하기
      -------------------------------------------
      #include <time.h>

      char *ctime(const time_t *timep);

      * timep: 초 단위 시간 정보를 저장한 주소
      -------------------------------------------
      초 단위 시간을 인자로 받아 사람이 보기 편한 형태로 변환해 문자열로 리턴함.
      요일 월 일 시:분:초 연도

      asctime(): tm 구조체 시간을 변환하여 출력하기
      -------------------------------------------
      #include <time.h>

      char *asctime(const struct tm *tm);

      * tm: 시간 정보를 저장한 tm 구조체 주소
      -------------------------------------------
      tm 구조체로 분해된 시간을 인자로 받고 사람이 보기 편한 형태로 변경해 문자열로 리턴한다.

      strftime(): 출력 형식 기호를 사용하여 출력하기
      위의 함수들은 출력형식을 사용자가 마음대로 지정할 수 없는데, 이 함수를 사용하면 printf문처럼 형식 지정자를 사용해 시간 정보를 출력할 수 있다.
      -------------------------------------------
      #include <time.h>

      size_t strftime(char *s, size_t max, const char *format, const struct tm *tm);

      * s: 출력할 시간 정보를 저장할 배열 주소
      * max: 배열 s의 크기
      * format: 출력 형식을 지정한 문자열
      * tm: 출력할 시간 정보를 저장한 구조체 주소
      -------------------------------------------
      tm 구조체에 저장된 시간 정보를 format에 지정한 출력 형식에 따라 변환해 s가 가리키는 배열에 저장한다. max는 배열 s의 크기이고, formant에 사용할 수 있는 시간 정보 출력 형식 지정자는 다음과 같다.

      | 기호 | 기능 | 기호 | 기능 |
      | &a | 지역 시간대의 요일명 약자 | %A | 지역 시간대의 요일명 |
      | %b | 지역 시간대의 월 이름 약자 | %B | 지역 시간대의 월 이름 |
      | %c | 지역 시간대에 적합한 날짜와 시간 표시 형태 | %C | 세기를 두 자리로 표시(연도/100) |
      | %d | 날짜(1~31) | %D | 날짜(%m/%d/%y) |
      | %e | 날짜(1~31). 한 자릿수는 앞에 공백 추가 | %F | %Y-%m-%d 형태로 표현 |
      | %g | 연도(00~99) | %G | 연도(0000~9999) |
      | %h | 지역 시간대 월 이름 역자 | %j | 일 년 중 날 수(001~365) |
      | %H | 24시간 기준 시간(00~23) | %l | 12시간 기준 시간(01~12) |
      | %k | 24시간 기준 시간(0~23). 한 자릿수는 앞에 공백 추가 | %I | 12시간 기준 시간(1~12), 한 자릿수는 앞에 공백 추가 |
      | %m | 월(01~12) | %M | 분(00~59) |
      | %p | AM. PM 표시 | %P | am, pm 표시 |
      | %r | 시간 표시하고 %p 수행 | %R | %H:%M 형태로 시간 표시 |
      | %s | 1970.1.1 00:00:00 기준으로 현재 시간을 초로 표시 | %S | 초를 10진수로 표시(00~60) |
      | %t | 탭 추가 | %T | %H:%M:%S 형태로 시간 표시 |
      | %n | 줄 바꿈 | %U | 연중 주간 수 표시(00~53), 첫번째 일요일을 01주의 첫날로 설정 |
      | %u | 요일(1~7, 1은 월요일) | %V | ISO 8601 표준으로 연중 주간 수 표시(01~53) |
      | %w | 요일(0~6, 0은 일요일) | %W | 연중 주간 수 표시(01~53), 첫번째 월요일을 01주의 첫날로 설정 |
      | %x | 지역 시간대에 적합한 날짜 표시 | %X | 지역 시간대에 적합한 시간 표시 |
      | %y | 연도(00~99) | %Y | 네 자릿수 연도 |
      | %z | UTC와의 시차 표시 | %Z | 시간대 이름 약자 |
