## ERROR too many open files 대처
### 배경
Kafka broker 5 대 중 3 대가 갑자기 process killed 되는 이슈가 일어났다. (~~ES 한 숨 돌리니 Kafka 가 말썽...~~)  
로그를 살펴보니 아래와 같은 에러를 발견.

* **/var/log/syslog**
```  
Nov 29 21:48:39 <HOSTNAME> kafka-server-start.sh[106713]: [2021-11-29 21:48:39,800] INFO [ReplicaManager broker=1] Broker 1 stopped fetcher for partitions <TOPIC LIST> because they are in the failed log dir <LOG DIR PATH> (kafka.server.ReplicaManager)
Nov 29 21:48:39 <HOSTNAME> kafka-server-start.sh[106713]: [2021-11-29 21:48:39,801] INFO Stopping serving logs in dir <LOG DIR PATH> (kafka.log.LogManager)
Nov 29 21:48:39 <HOSTNAME> kafka-server-start.sh[106713]: [2021-11-29 21:48:39,801] FATAL Shutdown broker because all log dirs in <LOG DIR PATH> have failed (kafka.log.LogManager)
```
### 원인
Kafka 와 Zookeeper 는 데이터 저장 경로가 기본적으로 /tmp 디렉토리 아래로 지정되어 있다고 한다.  
데이터가 OS 상에서 기본적으로 돌아가는 cron 에 의해 주기적으로 삭제되는 /tmp 디렉토리 아래에 데이터를 쌓는다는 말인 즉슨,  
파일 형태로 저장되는 Kafka 의 파티션 내 데이터에 대해 자동 삭제 등의 설정이 없다는 의미로 받아들여진다.
(확실치 않아서 찾아볼 필요 있음)

즉, Kafka 는 데이터 파일의 Open 개수에 크게 신경을 쓰지 않는다는(자동으로 Close 해주는 기능이 없다는) 것으로 해석할 수도 있을 것 같다.

실제로 Kafka 의 데이터 파일은 Close 되지 않는다.  
생각해보면 당연한게, 끊임없이 Pub/Sub 이 일어나는 파티션 파일이 close 된다는 건 말이 안된다.

이 과정에서 open 된 파일의 개수가 설정된 최대 open files 설정보다 많아졌고, 
높은 부하로 인해 too many open files 에러와 함께 Kafka 프로세스가 kill 되어버린 것이다.

### Open Files Limit
특정 프로세스/유저가 OS 에 요청할 수 있는 최대 open 가능한 파일 개수에는 한계치를 설정할 수 있다.  
여기서 그 한계치를 넘으면 too many open files 에러로 해당 프로세스/해당 유저가 사용하는 프로세스가 shutdown 되는 것이다.  
(**open 가능한 파일**이란, HTTP 나 JDBC 커넥션 등의 소켓을 포함한 파일의 개수를 의미한다)

한계치는 두 종류로 나뉜다.
* **Sort** : 한계치를 넘어도 경고 이메일 정도만 보내는 것으로 끝남. root 계정이 아닌 유저도 설정이 가능
* **Hard** : 설정 한계치를 넘을 수 없다. 한계치를 넘으면 too many open files 에러를 출력하며 프로세스 kill. root 계정에서만 설정이 가능


이 한계치는 위에서 말했듯 프로세스, 유저에 대해 설정할 수 있으며, 당연히 OS 전체 설정도 가능하다.

#### OS 전체 Limit 설정 확인
```
### 명령어로 확인
# ulimit -a

core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 64113
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1048576					### 이거!
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) unlimited
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited

### 파일로 확인
# cat /etc/security/limits.d/limits.conf

```
#### 프로세스별 Limit 설정 확인
```
# cat /proc/<PID>/limits
Limit                     Soft Limit           Hard Limit           Units
Max cpu time              unlimited            unlimited            seconds
Max file size             unlimited            unlimited            bytes
Max data size             unlimited            unlimited            bytes
Max stack size            8388608              unlimited            bytes
Max core file size        unlimited            unlimited            bytes
Max resident set          unlimited            unlimited            bytes
Max processes             257104               257104               processes
Max open files            1048576              1048576              files					### 이거!
Max locked memory         65536                65536                bytes
Max address space         unlimited            unlimited            bytes
Max file locks            unlimited            unlimited            locks
Max pending signals       257104               257104               signals
Max msgqueue size         819200               819200               bytes
Max nice priority         0                    0
Max realtime priority     0                    0
Max realtime timeout      unlimited            unlimited            us
```

### 작업
해당 Kafka 는 systemd 로 관리되는 프로세스이고, kafka 라는 계정으로 움직이고 있기 때문에, OS 의 root 계정으로 설정해 봐야 의미가 없었다.  
그래서 아예 kafka systemd unit file 에 아래와 같은 설정을 추가하였다.

```
[Service]
LimitAS=infinity          ### ulimit -v : 최대 가상 메모리 사이즈
LimitRSS=infinity         ### ulimit -m : 최대 메모리 사이즈
LimitCORE=infinity        ### ulimit -c : 최대 코어 파일 사이즈
LimitNOFILE=infinity      ### ulimit -n : 최대 오픈 파일 수
```
위처럼 unit file 에 추가해두면, 해당 service 를 시작할 때 limit 설정이 함께 들어가기 때문에 일일이 프로세스 limits 에 올릴 필요도 없고  
굳이 OS 쪽으로 꺾어서 시스템 전체 limit 설정을 가져올 필요도 없을 것이다.

open file 개수는 서버의 디스크, 메모리 사용량에 큰 영향을 주기 때문에 항상 설정에 주의해야 할 것 같다.

### 과제
Kafka 자체에서 file limit 관련 설정이 있다고 하는데... 뭔지 아직 못찾았다(server.properties 에 있다고는 하는데).

* 이런 방법도 있네? https://shanavas.org/kafka/2019/03/19/tracking-down-kafka-files.html
* Kafka 업그레이드(2.1.0 -> 2.1.1)로 해결? https://www.facebook.com/groups/kafka.kru/posts/704572363317423/

### 참고
* https://velog.io/@dev_osj/%EC%9E%A5%EC%95%A0-too-many-open-files
* https://shinwusub.tistory.com/130
