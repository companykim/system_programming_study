# chapter 9 메모리 매핑

  ## 1. 메모리 매핑과 해제

  메모리 매핑은 파일을 프로세스의 메모리에 매핑하는 것 </br>
  프로세스에 전달할 데이터를 저장한 파일을 직접 프로세스의 가장 주소 공간으로 매핑한다.
  read(), write() 함수를 사용하지 않고도 프로그램 내부에서 정의한 변수를 사용해 파일에서 데이터를 읽거나 쓸 수 있다.

  ### 메모리 매핑 함수
  
    mmap() 함수를 사용하면 파일을 프로세스의 가상 메모리에 매핑할 수 있다.
    매핑한 메모리의 위치와 크기를 인자로 받는다.

    mmap(): 메모리 매핑
    ----------------------------------------
    #include <sys/mman.h>

    void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);

    * addr: 매핑할 메모리의 주소
    * length: 메모리 공간의 크기
    * prot: 보호 모드
    * flags: 매핑된 데이터의 처리 방법 지정 상수
    * fd: 파일 기술자
    * offset: 파일 오프셋
    ----------------------------------------
    fd가 가리키는 파일에서 offset으로 지정한 오프셋부터 length 크기만큼 데이터를 읽어 addr이 가리키는 메모리 공간에 매핑한다.
    prot에는 읽기, 쓰기 등 보호 모드를 지정하고 flags에는 읽어온 데이터를 처리하기 위한 정보를 지정한다.

    prot에 사용할 수 있는 값은 상수로 정의되어 있으며, OR 연산자로 연결해 지정할 수 있다.
    PROT_READ: 매핑된 파일을 읽기만 한다.
    PROT_WRITE: 매핑된 파일에 쓰기를 허용한다.
    PROT_EXEC: 매핑된 파일을 실행할 수 있다.
    PROT_NONE: 매핑된 파일에 접근할 수 없다.

    prot에 PROT_WRITE를 지정하려면 flags 인자에 MAP_PRIVATE를 지정하거나 파일을 쓰기 가능 상태로 열어야 한다.

    flags에 지정할 수 있는 값도 상수로 정의되어 있다.
    MAP_SHARED: 다른 프로세스와 데이터 변경 내용을 공유한다. 이 플래그가 설정되어 있으면 쓰기 동작은 매핑된 메모리의 내용을 변경한다.
    MAP_SHARED_VALIDATE: MAP_SHARED와 같으나 전달받은 플래그를 커널이 모두 확인하고 모르는 플래그가 있을 경우 오류로 처리한다.
    MAP_PRIVATE: 데이터 변경 내용을 공유하지 않는다. 이 플래그가 설정되어 있으면 최초의 쓰기 동작에서 매핑된 메모리의 사본을 생성하고 매핑 주소는 사본을 가리킨다.
    MAP_ANONYMOUS: fd를 무시하고 할당된 메모리 영역을 0으로 초기화한다.
    MAP_ANON: MAP_ANONYMOUS와 동일한 플래그로 다른 시스템과의 호환성을 위해 제공한다.
    MAP_FIXED: 매핑할 주소를 정확히 지정한다. MAP_FIXED 플래그를 지정하고 mmap() 함수가 성공하면 해당 메모리 영역의 내용은 매핑된 내용으로 변경된다.
    MAP_NORESERVE: MAP_PRIVATE를 지정하면 시스템은 매핑에 할당된 메모리 공간만큼 스왑 영역을 할당한다. 이 스왑 영역은 매핑된 데이터의 사본을 저장하는데 사용한다. MAP_NORESERVE를 지정하면 스왑 영역을 할당하지 않는다.

  ### 메모리 매핑 해제 함수

    munmap(): 메모리 매핑 해제
    ----------------------------------------
    #include <sys/mman.h>

    int munmap(void *addr, size_t length);

    * addr: 매핑된 메모리의 시작 주소
    * length: 메모리 영역의 크기
    ----------------------------------------
    addr이 가리키는 영역에 length 크기만큼 할당해 매핑한 메모리를 해제한다. 즉, 메모리에 매핑된 [addr+length] 크기만큼 메모리 영역을 해제한다.
    munmap() 함수로 매핑을 해제한 메모리 영역에 접근하면 SIGSEGV 시그널이 발생한다.

  ### 메모리 매핑의 보호 모드 변경 함수

    mprotect(): 보호 모드 변경
    ----------------------------------------
    #include <sys/mman.h>

    int mprotect(void *addr, size_t len, int prot);

    * addr: 매핑된 메모리의 시작 주소
    * len: 메모리 영역의 크기
    * prot: 보호 모드
    ----------------------------------------
    매핑된 메모리의 보호 모드는 mmap() 함수로 메모리 매핑을 수행할 때 초기값을 설정한다.
    addr로 지정한 주소에 len 크기만큼 매핑된 메모리의 보호 모드를 prot에 지정한 값으로 변경한다. prot 인자에는 PROT_READ, PROT_WRITE, PROT_EXEC, PROT_NONE을 사용한다.

## 2. 파일 확장과 메모리 매핑
  존재하지 않는 파일이나 크기가 0인 파일은 mmap() 함수를 사용해 메모리에 매핑할 수 없다. </br>
  프로그램에서 빈 파일을 생성했다면 mmap() 함수로 메모리에 매핑시키기 전에 함수를 이용해 파일의 크기를 확장해야 한다.

    truncate(): 경로명을 사용한 파일 크기 확장
    ----------------------------------------
    #include <unistd.h>
    #include <sys/types.h>

    int truncate(const char *path, off_t length);

    * path: 크기를 변경할 파일의 경로
    * length: 변경하려는 크기
    ----------------------------------------
    path에 지정한 파일의 크기를 length로 지정한 크기로 변경할 수 있다.
    파일의 원래 크기가 length보다 크면 length 길이를 초과하는 부분은 버리고, length보다 작으면 파일의 크기를 증가시키고 해당 부분을 널 바이트('\0')로 채운다.
    경로로 지정한 파일에 대한 쓰기 권한이 있어야 한다.

    ftruncate(): 파일 기술자를 사용한 파일 크기 확장 
    ----------------------------------------
    #include <unistd.h>
    #include <sys/types.h>

    int ftruncate(int fd, off_t length);

    * fd: 크기를 변경할 파일의 파일 기술자
    * length: 변경하려는 크기
    ----------------------------------------
    파일의 경로명 대신 파일 기술자를 받는다. 즉, fd에 지정한 파일의 크기를 length로 지정한 크기로 변경한다. 이때 파일 기술자는 크기를 변경할 파일을 열고 리턴받은 값이어야 한다.
    일반 파일과 공유 메모리에만 사용할 수 있다.
    파일의 오프셋을 변경하지 않지만 파일의 상태 정보에서 st_ctime과 st_mtime을 수정한다. 
    ftruncate() 함수로 디렉토리에 접근하거나 쓰기 권한이 없는 파일에 접근하면 오류가 발생한다.

    mremap(): 메모리 주소를 다시 매핑하기
    ----------------------------------------
    #define _GNU_SOURCE
    #include <sys/types.h>

    void *mremap(void *old_address, size_t old_size, size_t new_size, int flags);

    * old_address: 크기를 변경할 메모리 주소
    * old_size: 현재 메모리 크기
    * new_size: 바꾸려는 메모리 크기
    * flags: 0 또는 MREMAP_MAYMOVE
    ----------------------------------------
    리눅스에서만 제공하며, 매핑된 메모리의 크기와 위치를 변경할 수 있다. old_address에 현재 매핑된 주소를 지정하고 old_size에는 현재 메모리 크기를 지정한다.
    new_size에는 다시 매핑할 메모리 크기를 지정한다. flags에는 0을 지정하거나 MREMAP_MAYMOVE를 지정한다.
    MREMAP_MAYMOVE를 지정하면 크기를 변경할 때 매핑의 위치를 이동해도 된다는 뜻이다.

## 3. 매핑된 메모리 동기화와 데이터 교환

  시스템은 메모리에 매핑된 파일의 내용을 백업 공간에 복사해둔다. 따라서 매핑된 메모리의 내용과 백업 내용이 일치하도록 동기화해야 한다. </br>
  메모리 매핑 기능을 사용해 매핑한 데이터를 부모 프로세스와 자식 프로세스가 공유하도록 할 수 있다. </br>
  공유를 사용하면 부모 프로세스와 자식 프로세스가 데이터를 주고 받을 수 있다.

  ### 매핑된 메모리의 동기화 함수
    매핑된 메모리 영역과 백업 저장 장치의 내용이 일치하도록 동기화하는데 msync() 함수를 사용한다.

      msync(): 매핑된 메모리 동기화
      ----------------------------------------
      #include <sys/mman.h>

      int msync(void *addr, size_t length, int flags);

      * addr: 매핑된 메모리의 시작 주소
      * length: 메모리 영역의 크기
      * flags: 동기화 동작
      ----------------------------------------
      인자로 지정한 addr로 시작하는 메모리 영역에서 [addr+length]만큼의 내용을 백업 저장 장치로 기록한다.
      이 함수를 사용하지 않으면 munmap() 함수가 호출되기 전에 변경된 메모리 내용이 저장되는지 보장할 수 없다.
      flags는 msync() 함수의 동작을 지시하는 값이다.
      MS_ASYNC: 비동기 쓰기 작업을 수행한다. msync() 함수는 즉시 리턴하고, 함수가 리턴한 후 적절한 시점에 쓰기 작업을 수행한다.
      MS_SYNC: 쓰기 작업을 완료할 때까지 msync() 함수가 리턴하지 않는다. 메모리의 크기가 클 경우 비교적 시간이 걸릴 수 있다.
      MS_INVALIDATE: 메모리에 있는 기존 내용을 무효화할 것인지 확인한다.

    ### 메모리 매핑을 사용한 데이터 교환
      메모리 매핑 함수를 사용하면 매핑된 파일을 부모 프로세스와 자식 프로세스가 공유하도록 만들 수 있다.
      즉, 부모 프로세스나 자식 프로세스가 매핑된 메모리의 내용을 변경하면 다른 프로세스도 변경 내용을 알 수 있다.
