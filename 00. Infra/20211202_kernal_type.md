## OS 에 따른 커널 구조의 차이

실장님이 인프라팀쪽 면접 진행할 때마다 꼭 묻는 게 커널에 대한 질문이어서 가볍게 정리해봄(나도 언젠간...)

### OS 와 커널
* **OS** : 운영체제. 서버 위에서 소프트웨어들이 움직일 수 있도록 제어, 하드웨어와 소프트웨어 사이에서 프로그램들을 관리할 수 있는 소프트웨어의 일종. 미들웨어라고도 하는듯?
* **커널** : OS 의 핵심적인 구성 요소로, 해당 서버에서 현재 움직이고 있는 소프트웨어(어플리케이션?)에 필요한 서비스와 리소스를 관리하는 것 컴퓨터의 자원들만을 관리하는 역할

### 커널의 구조에 따른 유형
OS 의 일반적인 기능을 커널 모드에 넣느냐, 유저 모드에 넣느냐에 따라 모놀리식(단일)과 마이크로 커널로 나뉜다.  
시스템 프로세서는 커널 모드와 유저 모드라는 멀티 모드로 구성됨을 통해 전체 시스템을 보호한다.

* 커널 모드 : 모든 시스템 메모리 및 CPU 에 접근 가능하며, 유저 모드로 접근하는 건 제한된다
* 유저 모드 : 일반적인 사용자 어플리케이션이 여기에서 실행된다
* 시스템 콜 : 유저 모드에 있는 어플리케이션이 커널 모드로 모드를 변경한 상태에서 OS 운영체제 서비스를 호출하는 것

#### 단일(모놀리식) 커널
- UNIX, Linux 등에서 사용
- 커널의 핵심 기능을 구현하는 모듈이 구분되지 않고 하나로 구성되어 있다
- 운영 체제의 일반적인 기능을 커널과 동일한 메모리 공간에 적재해서 사용 및 실행하는 방법
- 즉, 어플리케이션을 제외한 모든 시스템 서비스들을 커널이 직접 처리한다.
- 커널의 크기는 상대적으로 크지만, 커널 내부에서 서비스들이 시스템 자원을 공유하며 효율적으로 관리된다

#### 마이크로 커널
- Windows, MacOS 등에서 사용
- 핵심적인 기능만을 커널에 담고, 기존 단일 커널에서 처리되던 시스템 서비스(VFS, Application IPC, Device Driver) 는 커널 위에서 개별적인 서버 형태로 존재하며, 마치 데몬 프로그램처럼 실행된다
- 즉, OS 의 시스템 서비스를 유저 모드에서 처리한다(핵심 서비스만 커널 모드에서 처리)
- 가볍고, 커널의 보안과 안정성을 향상시킬 수 있다
- ※ 데몬 : 멀티태스킹 OS 에서 사용자가 직접적으로 제어하지 않고, 백그라운드에서 움직이며 작업을 처리하는 프로그램
- 스템 복잡도가 높아질수록 시스템 부하(오버헤드) 가 높아진다

#### Hybrid Kernel(Windows NT)
- windows NT 는 초기에 마이크로 커널로서 개발되었지만, 주요 시스템 서비스가 커널 모드에서 동작한다는 점에서는 단일 커널의 특징을 갖고 있어 Hybrid Kernel 이라고 불린다 한다.

### 참고
* 공부를 위해 정독 필요 : https://5equal0.tistory.com/entry/Linux-Kernel-%EC%BB%A4%EB%84%90%EC%9D%98-%EA%B0%9C%EB%85%90%EA%B3%BC-%EC%BB%A4%EB%84%90%EC%9D%98-%EA%B5%AC%EC%A1%B0
* 참고 : https://blog.naver.com/PostView.nhn?blogId=cjsksk3113&logNo=222238471338
* 레드햇 홈페이지에서 잘 정리해줌! : https://www.redhat.com/ko/topics/linux/what-is-the-linux-kernel

### 과제
OS 최적화를 위한 커널 환경설정 튜닝을 해 본 것이 있다면?
