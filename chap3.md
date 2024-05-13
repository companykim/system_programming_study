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
  | S_IFSOCK | 0140000 | 소켓 파일 |
  | S_IFLINK | 0120000 | 심벌릭 링크 파일 |
  | S_IFREG | 0100000 | 일반 파일 |
  | S_IFBLK | 0060000 | 블록 장치 특수 파일 |
  | S_IFDIR | 0040000 | 디렉토리 |
  | S_IFCHR | 0020000 | 문자 장치 특수 파일 |
  | S_IFIFO | 0010000 | FIFO 파일 |

  매크로를 이용한 파일 종류 검색
  | 매크로명 | 매크로 정의 | 기능 |
  | :--: | :--: | :--: |
  | S_ISLNK(m) | (((m) & S_IFMT) == S_IFLNK)| 참이면 심벌릭 링크 파일 |
  | S_ISREG(m) | (((m) & S_IFMT) == S_IFREG) | 참이면 일반 파일 |
  | S_ISDIR(m) | (((m) & S_IFMT) == S_IFDIR) | 참이면 디렉토리 |
  | S_ISCHR(m) | (((m) & S_IFMT) == S_IFCHR) | 참이면 문자 장치 특수 파일 |
  | S_ISBLK(m) | (((m) & S_IFMT) == S_IFBLK) | 참이면 블록 장치 특수 파일 |
  | S_ISFIFO(m) | (((m) & S_IFMT) == S_IFIFO) | 참이면 FIFO 파일 |
  | S_ISSOCK(m) | (((m) & S_IFMT) == S_IFSOCK) | 참이면 소켓 파일 | 
  
  각 매크로는 인자로 받은 mode 값을 0xF000과 AND 연산한다. </br>
  AND 연산의 결과를 파일의 종류별로 정해진 값과 비교해 참인지 여부로 파일의 종류를 판단한다.


  ### 파일 접근 권한 검색

  상수를 이용한 파일 접근 권한 검색
  | 상수명 | 상숫값 | 기능 |
  | :--: | :--: | :--: |
  | S_ISUID | 0x800 | st_mode 값과 AND 연산으로 setuid 설정 확인 |
  | S_ISGID | 0x400 | st_mode 값과 AND 연산으로 setgid 설정 확인 |
  | S_ISVTX | 0x200 | st_mode 값과 AND 연산으로 sticky  비트 설정 확인 |
  | S_IREAD | 00400 | st_mode 값과 AND 연산으로 소유자의 읽기 권한 확인 |
  | S_IWRITE | 00200 | st_mode 값과 AND 연산으로 소유자의 쓰기 권한 확인 |
  | S_IEXEC | 00100 | st_mode 값과 AND 연산으로 소유자의 실행 권한 확인 | 

  읽기/쓰기/실행 권한을 추출하는 상숫값이 8진수라는 점에 주의한다. </br>
  st_mode의 값을 왼쪽으로 3비트 이동시키거나 상숫값을 오른쪽으로 3비트 이동시킨 후 AND 연산을 수행하면 그룹의 접근 권한을 알 수 있다.

  파일의 접근 권한 검색 상수(POSIX)
  | 상수명 | 상숫값 | 기능 |
  | :--: | :--: | :--: |
  | S_IRWXU | 00700 | 소유자에게 읽기/쓰기/실행 권한 |
  | S_IRUSR | 00400 | 소유자에게 읽기 권한 |
  | S_IWUSR | 00200 | 소유자에게 쓰기 권한 |
  | S_IXUSR | 00100 | 소유자에게 실행 권한 |
  | S_IRWXG | 00070 | 그룹에게 읽기/쓰기/실행 권한 |
  | S_IRGRP | 00040 | 그룹에게 읽기 권한 |
  | S_IWGRP | 00020 | 그룹에게 쓰기 권한 |
  | S_IXGRP | 00010 | 그룹에게 실행 권한 |
  | S_IRWXO | 00007 | 기타 사용자에게 읽기/쓰기/실행 권한 |
  | S_IROTH | 00004 | 기타 사용자에게 읽기 권한 |
  | S_IWOTH | 00002 | 기타 사용자에게 쓰기 권한 |
  | S_IXOTH | 00001 | 기타 사용자에게 실행 권한 |

    access(): 함수를 이용한 접근 권한 검색 </br>
    -----------------------------------------
    #include <unistd.h>

    int access(const char *pathname, int mode);

    * pathname: 접근 권한을 알고자 하는 파일의 경로
    * mode : 접근 권한
    -----------------------------------------
    pathname에 지정된 파일이 mode로 지정한 권한을 지니고 있는지 확인하고 리턴한다.
    단, 유효사용자 ID(EUID)가 아닌 실제 사용자 ID(RUID)에 대한 접근 권한만 확인할 수 있다.
    mode에 사용할 수 있는 상수
    R_OK: 읽기 권한 확인
    W_OK: 쓰기 권한 확인
    X_OK: 실행 권한 확인
    F_OK: 파일이 존재하는지 확인

  ### 파일 접근 권한 변경
    명령은 chmod이다.

    chmod(): 파일명으로 접근 권한 변경
    -----------------------------------------
    #include <sys/stat.h>

    int chmod(const char *pathname, mode_t mode);

    * pathname: 접근 권한을 변경하려는 파일의 경로
    * mode: 접근 권한
    -----------------------------------------
    접근 권한을 변경할 파일의 경로를 받아 mode에 지정한 상숫값으로 권한을 변경한다.
    특수 접근 권한을 변경할 때는 S_ISUID, S_ISGID, S_ISVTX를 이용함.
    기존 접근 권한과 관계없이 접근 권한을 모두 새로 지정하려면 상수를 이용해 권한을 생성한 후 인자로 지정함.

    fchmod(): 파일 기술자로 접근 권한 변경
    -----------------------------------------
    #include <sys/stat.h>

    int fchmod(int fd, mode_t mode);

    * fd: 열려 있는 파일의 파일 기술자
    * mode: 접근 권한
    -----------------------------------------
    접근 권한을 변경할 파일의 파일 기술자를 받아 mode에 미리 정의한 상숫값으로 변경할 권한을 지정함.
    chmod()는 파일의 경로를 받지만 fchmod()는 이미 열려 있는 파일의 파일 기술자를 받아 접근 권한을 변경한다.

## 3. 링크 파일 생성
  링크: 이미 있는 파일이나 디렉토리에 접근할 수 있는 새로운 이름을 의미함. </br>
  같은 파일이나 디렉토리지만 여러 이름으로 접근할 수 있게 하는 것. </br>
  이전 시스템과의 호환성을 유지하거나 경로가 복잡한 파일에 간단한 경로를 제공하는 등 사용자 편의성을 높일 수 있다.

  ### 하드 링크
    파일에 접근할 수 있는 파일명을 새로 생성하는 것.

    link(): 하드 링크 생성
    -----------------------------------------
    #include <unistd.h>

    int link(const char *oldpath, const char *newpath);

    * oldpath: 기존 파일의 경로
    * newpath: 새로 생성할 링크의 경로
    -----------------------------------------
    기존 파일의 경로를 첫번째 인자인 oldpath로 받고, 새로 생성할 하드 링크 파일의 경로를 두번째 인자인 newpath로 받는다.
    하드 링크는 같은 파일 시스템에 있어야 하므로 두 경로를 반드시 같은 파일 시스템으로 지정해야 한다.

  ### 심벌릭 링크
    기존 파일에 접근할 수 있는 다른 파일을 만든다. 기존 파일과 다른 inode를 사용하며, 기존 파일의 경로를 저장한다.

    symlink(): 심벌릭 링크 생성
    -----------------------------------------
    #include <unistd.h>

    int symlink(const char *target, const char *linkpath);

    * target: 기존 파일의 경로
    * linkpath: 새로 생성할 심벌릭 링크의 경로
    -----------------------------------------
    기존 파일의 경로를 첫번째 인자인 target으로 받고 새로 생성할 심벌릭 링크의 경로를 두번째 인자인 linkpath로 받는다.
    기존 파일과 다른 파일 시스템에도 생성할 수 있다.

    lstat(): 심벌릭 링크의 정보 검색
    -----------------------------------------
    #include <sys/types.h>
    #include <sys/stat.h>
    #include <unistd.h>

    int lstat(const char *pathname, struct stat *statbuf);

    * pathname: 심벌릭 링크의 경로
    * statbuf: 새로 생성할 링크의 경로
    -----------------------------------------
    심벌릭 링크 자체의 파일 정보를 검색함.
    stat() 함수로 검색하면 원본 파일에 대한 정보가 검색된다는 점에 주의해야 함.

    readlink(): 심벌릭 링크의 내용 읽기
    -----------------------------------------
    #include <unistd.h>

    ssize_t readlink(const char *pathname, char *buf, size_t bufsiz);

    * pathname: 심벌릭 링크의 경로
    * buf: 읽어온 내용을 저장할 버퍼
    * bufsiz: 버퍼의 크기
    -----------------------------------------
    심벌릭 링크 자체의 데이터를 읽는다. 심벌릭 링크의 경로를 받아 해당 파일의 내용을 읽는다.
    호출시 읽은 내용을 저장할 버퍼와 해당 버퍼의 크기를 인자로 지정한다.

    realpath(): 심벌릭 링크 원본 파일의 경로 읽기
    -----------------------------------------
    #include <limits.h>
    #include <stdlib.h>

    char *realpath(const char *path, char *resolved_path);

    * path: 심벌릭 링크의 경로명
    * resolved_path: 경로명을 저장할 버퍼 주소
    -----------------------------------------
    심벌릭 링크명을 받아 실제 경로명을 resolved_name에 저장한다.
    심벌릭 링크가 가리키는 원본 파일의 실제 경로명을 확인하는 용도이다.

  ### 링크 끊기
  
    unlink(): 링크 끊기
    -----------------------------------------
    #include <unistd.h>

    int unlink(const char *pathname);

    * pathname: 삭제할 링크의 경로
    -----------------------------------------
    unlink() 함수에서 연결을 끊은 경로명이 그 파일을 참조하는 마지막 링크라면 파일은 삭제된다.
    만약 인자로 지정한 경로명이 심벌릭 링크이면 링크가 가리키는 원본 파일이 아니라 심벌릭 링크 파일이 삭제된다.
