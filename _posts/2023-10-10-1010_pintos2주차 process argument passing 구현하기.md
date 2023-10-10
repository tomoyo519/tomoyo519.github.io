### process 생성을 위한 argument passing(인자전달)


##### Process 생성을 위한 Argument passing ( 인자 전달 )

- Argument passing 의 이유와 목적
  하는 이유 : 예를들어 `/bin/ls -l foo bar` 와 같은 명령어를 입력 받았을 때, 단어를 띄어쓰기 별로 쪼개어 스택에 넣어야 한다. 그래야 각각의 명령어를 커널에 전달 할 수 있기 때문.
  목적 : passing 한 텍스트들은 각각의 스택의 포인터로 argv의 원소에 저장하여 전달 해야 한다.

- 스택 저장 방법
  %rdi, $rsi, %rdx, %rcx, %r8, $r9 시퀀스들을 전달 하기 위해 정수 레지스터를 사용한다.

- ![스크린샷 2023-10-10 21.55.21](https://p.ipic.vip/shla0f.jpg)

- 스택에 넣을 떄 주의할 점

  1. argv[i] 값을 넣을 때는 마지막을 항상 null값('/0')을 넣기

  2. 값을 저장한 주소를 미리 저장해둔다

  3. 값을 저장 후 8바이트로 맞추기 위해 패딩을 넣는다.

     (x86 아키텍처는 데이터를 효율적으로 처리하기 위해 일반적으로 4바이트 또는 8바이트(64비트 시스템에서) 경계에 데이터를 정렬함. 때문에 load 함수에서 실행파일을 메모리에 로드할때, 각 섹션의 크기가 8의 배수가 되도록 패딩을 추가함. 이렇게 하면 해당 섹션을 읽어올 때 CPU가 한번에 처리할 수 있는 단위로 데이터를 가져오므로, 메모리 접근 횟수를 줄이고 성능을 개선할 수 있음.)

  4. 스택은 아래로 증가한다.



- 테스트 코드로 코드 흐름을 파악할것. 예시 :open-normal

  - 테스트케이스에서 어떤 테스트케이스인지 main()으로 보냄. 이경우 open-normal
  - 해당 메인 함수의 위치 : `tests/main.c`
  - 각각의 test_main()으로 이동하여 시스템 콜을 수행

  ```
  #include <random.h>
  #include "tests/lib.h"
  #include "tests/main.h"

  int
  main (int argc UNUSED, char *argv[]) 
  {
    test_name = argv[0];

    msg ("begin");
    random_init (0);
    test_main ();
    msg ("end");
    return 0;
  }

  ```

  ​

  - tests/userporg/open-normal.c

    - open() 에 인자를 담아 시스템 콜을 요청.
      해당 요청은 `lib/user/syscall.c  `  로 이동.

  - lib/user/syscall.c
    해당 인자들은 인터럽트 프레임으로 담아 `userprog/syscall.c` 로 전달된다

    ```
    ​```c
    int
    open (const char *file) {
    	return syscall1 (SYS_OPEN, file);
    }
    ​```
    ```

    ​

  - userprog/syscall.c

    ```c
    ​```c
    void
    syscall_handler (struct intr_frame *f UNUSED) {
    	// TODO: Your implementation goes here.
    	uint64_t syscall_no = f->R.rax;  // 콜 넘버

    	// uint64_t a1 = f->R.rdi;		// 파일 네임
    	// uint64_t a2 = f->R.rsi;		// v(데이터)
    	// uint64_t a3 = f->R.rdx;      // 사이즈
    	// uint64_t a4 = f->R.r10;
    	// uint64_t a5 = f->R.r8;
    	// uint64_t a6 = f->R.r9;

    	...

    }
    ​```
    ```

     인터럽트 프레임(struct intr_frame *f) 으로 넘어온 사항은 다음과 같이 분리할 수 있다.

    - rax : 시스템 콜 넘버, 최종 값을 리턴받는 주소
    - rdi : file name 이 들어있고 첫번째 인자이다.
    - rsi : 값이 저장 되어있는주소가 시작되는 주소를 갖고 있고 두번쨰 인자이다.
    - rdx : 명령어 사이즈, 세번째 인자이다.

