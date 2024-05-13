# 파일 다루기

  ## 1. 파일 정보 검색

    파일 정보를 검색하려면 ls 명령을 사용한다.

    stat(): 파일명으로 파일 정보 검색
    -----------------------------------------
    #include <sys/types.h>
    #include <sys/stat.h>
    #include <unistd.h>

    int stat(const char *pathname, struct stat *statbuf);

    * pathname: 정보를 알고자 하는 파일명
    * statbuf: 검색한 파일 정보를 저장할 구조체 주소
    -----------------------------------------
    pathname에 지정한 파일의 정보를 검색한다. 
    이 함수로 파일의 정보를 검색할 때 파일에 대한 읽기/쓰기/실행 권한이 반드시 있어야 하는 것은 아니다.
    파일에 이르는 경로의 각 디렉토리에 대한 읽기 권한은 있어야 한다.

    stat 구조체
    stat() 함수로 검색한 inode 정보는 stat 구조체에 저장되어 리턴된다. 
    stat 구조체의 세부 구조는 man -s 2 stat으로 확인 가능.
    -----------------------------------------
    struct stat {
      dev_t            st_dev; 파일이 저장되어 있는 장치의 번호를 저장한다.
      ino_t            st_ino; 파일의 inode 번호를 저장한다.
      mode_t           st_mode; 파일의 종류와 접근 권한을 저장한다.
      nlink_t          st_nlink; 하드 링크의 개수를 저장한다.
      uid_t            st_uid; 파일 소유자의 UID를 저장한다.
      gid_t            st_gid; 파일 소유 그룹의 GID를 저장한다.
      dev_t            st_rdev; 장치 파일이면 주 장치 번호와 부 장치 번호를 저장한다. 장치 파일이 아니면 아무 의미가 없다.
      off_t            st_size; 
      bliksize_t       st_blksize; 파일 내용을 입출력할 때 사용하는 버퍼의 크기를 저장한다. 
      blkcnt_t         st_blocks; 파일에 할당된 파일 시스템의 블록 수로, 블록의 크기는 512 바이트이다.
      struct timespec  st_atim;
      struct timespec  st_mtim;
      struct timespec  st_ctim;

      #define st_atime  st_atim.tv_sec  마지막으로 파일을 읽거나 실행한 시각을 timespec 구조체로 저장한다. 저장 가능한 시각은 1970년 1월 1일 이후로 한다.
      #define st_mtime  st_mtim.tv_sec  마지막으로 파일의 내용을 변경(쓰기)한 시각을 저장한다. 이 값도 timespec 구조체로 저장한다. 
      #define st_ctime  st_ctim_tv_sec  마지막으로 inode의 내용을 변경한 시각을 저장한다. inode의 내용은 소유자/그룹 변경, 파일 크기 변경, 링크 개수 변경 등을 수행할 때 변경된다.
    };

    struct timespec {
      __kernel_time_t  tv_sec;    /* 초 (seconds) */
      long             tv_nsec;   /* 나노초 (nanoseconds) */
    };
    -----------------------------------------

    fstat(): 파일 기술자로 파일 정보 검색
    -----------------------------------------
    #include <sys/types.h>
    #include <sys/stat.h>
    #include <unistd.h>

    int fstat(int fd, struct stat *statbuf);

    * fd: 열려 있는 파일의 파일 기술자
    * statbuf: 검색할 파일 정보를 저장할 구조체 주소
    -----------------------------------------
    파일 경로 대신 현재 열려 있는 파일의 파일 기술자를 인자로 받아 파일 정보를 검색한 후 statbuf로 지정한 구조체에 저장한다.

  ## 2. 파일 접근 권한 제어
    stat 구조체의 st_mode 항목에는 파일의 종류와 접근 권한 정보가 저장된다.
    st_mode 항목의 값을 해석하려면 sys/stat.h 파일에 정의된 상수와 매크로를 이용해야 함.

  ### st_mode의 구조
    st_mode 항목의 데이터형인 mode_t는 unsigned int로 정의되어 있다.

  ### 파일의 종류 검색

    상수를 이용한 파일 종류 검색
    | 상수명 | 상숫값(8진수) | 기능 |
    | :--: | :--: | :--: |
    | S_IFMT | 0170000 | 파일의 종류 비트를 가져오기 위한 비트 마스크 |
    
