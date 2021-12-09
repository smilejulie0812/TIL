## JVM 버전과 auto heap sizing, swap off 설정의 관걔성

오늘 오전에 ES 7 이상부터 패키지 설치할 때 번들 JDK 가 함께 딸려오고, 사용할 JAVA 설정을 추가로 넣어주지 않는 한 그 번들 JDK 를 사용한다는 글을 올렸다.
그 내용의 후속편 같은 글로, auto heap sizing 기능을 swap 기능과 연관지어 정리해 보고자 한다.

우선...

### Auto Heap Sizing 이란?

Elasticsearch 를 구성함에 있어 가장 까다로운 절차라고 한다면 역시 Heap Size 설정일 것이다.
일반적으로 ES 의 Java Heap Size 는 다음의 사항을 고려하여 설정한다.
* OS 및 lucene 이 사용할 메모리 사이즈를 고려하여 OS 메모리의 반을 잡아준다
* heap size 조정 작업을 하면서 성능 저하가 일어날 것을 고려하여 xms 와 xmx 를 되도록 같게 설정한다
* (Java 움직임의 어떤 원리에 의해... 이해하면 추가하겠음) 32GB 를 초과하지 않도록 설정한다

그런데 ES 버전 7 이상부터, 이 Heap Size 를 일부러 고정시키지 않아도 사용 가능한 기능이 추가되었다고 한다!
바로 Auto Heap Sizing 기능이 그것이며, Java 14 이상 버전부터 사용 가능하다(ES 7 설치시 번들 JDK 가 버전 14 이상이므로 문제 없음)

ES 의 jvm.options 파일에서 Heap Size 를 고정시키지 않아도,
ES 자체에서 해당 노드에 부여된 역할(master, data, ingest 등)과 OS 전체 메모리 사용 현황을 고려햐여 실시간 ES 에 알맞은 Heap Size 로 메모리를 사용할 수 있는 기능이다.

이게 사용 가능해지면 ES 전용으로 Heap Size 를 고정시키지 않으므로 OS 측면에서 봤을 땐 낭비되는 메모리가 없어 효율적인 운영이 가능할 것이다.

그럼 여기서 이야기 방향성을 조금 틀어서,

### bootstrap.memory_lock 과 Swap 설정

Swap(스왑)이란 OS 에서 작업을 처리하는데 메모리를 초과해서 사용할 경우, 디스크에 일정 부분 잡아둔 사이즈(공간)만큼 메모리를 대신해서 사용하는 기능을 말한다.
물론 메모리에 비해 디스크는 처리 속도가 현저히 느리므로, 스왑을 사용하게 될 경우 해당 작업 처리 성능 속도는 말을 잇지 못할 정도일 것이다.
이 스왑 메모리로 전환되거나 다시 메모리를 사용하게 되는 과정에서도 부하가 걸리므로(디스크 스레싱!), 여러 의미로 이 스왑 기능을 설정한다는 것은 성능 면에서 피해야 할 부분이다.
(물론 성능은 둘째치고 스왑 메모리를 사용하더라도 프로세스를 돌려야 할 경우에는 스왑 기능을 사용하도록 설정하는 경우도 존재한다.)

추가) 어... 근데 Elastic 사는 이렇게 말하네
```
Swapping is very bad for performance, for node stability, and should be avoided at all costs. It can cause garbage collections to last for minutes instead of milliseconds and can cause nodes to respond slowly or even to disconnect from the cluster. In a resilient distributed system, it’s more effective to let the operating system kill the node.
```

참고로, Elastic 사 측은 Elasticsearch 사용시 스왑 기능을 사용하지 않는 것을 추천하고 있으며, 그 방법은 세 가지를 권장한다.

* OS 자체의 스왑 기능을 끄는 두 가지 방법
이 내용은 이미 내 블로그에 정리해 둔 내용이 있어 링크만 걸어둬야지
https://smilejulie0812.github.io/linux/swapmemory/

* elasticsearch.yml 에서 스왑 기능 끄기
```yaml
bootstrap.memory_lock: true
```
이 기능은 Elasticsearch 자체에서 설정해 둔 힙 메모리가 스왑되지 않도록 설정하는 방법이다.
(Linux 에서는 mLockall 을, Windows 에서는 VirtualLock 을 사용해서, 프로세스 주소 공간을 RAM 으로 잠가둔다고 한다 <-이 부분은 좀 더 이해가 필요)

해당 기능을 활성화시킨 후 Elasticsearch 를 시작하면, 아래 API 로 설정이 제대로 먹혔는지 확인할 수 있다.
```bash
GET _nodes?filter_path=**.mlockall
```

위의 기능을 추가했는데 mlockall 이 false 면 제대로 Swap-off 가 처리되지 않았다는 의미이다.
이런 경우는 보통 Elasticsearch 실행 유저에게 메모리를 잠글 수 있는 mlockall 권한이 없기 때문이므로, 아래 방법을 참고해서 유저에게 권한을 부여해주자.

아카이브 파일로 설치한 경우
```bash
vi /etc/security/limits.conf
# allow user 'elasticsearch' mlockall
elasticsearch soft memlock unlimited
elasticsearch hard memlock unlimited
```

패키지 파일로 설치한 경우
```bash
vi /usr/lib/systemd/system/elasticsearch.service
[Service]
LimitMEMLOCK=infinity
```

이래도 안되는 경우에는 limits.xonf 설정값이 반영되지 않아서 그렇다고 하니 여기까지 해 주자
```bash
ulimit -l
unlimited
```

혹시 그래도 안되면 이런 것도 시도해 보자(...)
```bash
export ES_JAVA_OPTS="$ES_JAVA_OPTS -Djna.tmpdir=<path>"
./bin/elasticsearch
```

중간에 설정 수순서처럼 적어버렸는데 요점은, 스왑을 끄는 기능(Swap-off)을 사용한다는 것은,
설정한 메모리보다 더 많은 메모리를 사용하게 될 때, Swap 메모리를 쓸 수 없게 되므로, JVM 이 종료될 가능성이 커지게 된다(이게 뭐다 지긋지긋한 OutOfMemory 되시겠다). 

자 그렇다면 이제 정리해 보자. 하고 싶었던 말이 뭐냐면,

### 결과적으로...
Swap-off 되어있지 않은 서버에 ES 를 돌릴 때 Auto Heap Sizing 으로 설정하면 아마도,
ES 가 필요한 만큼 탄력적으로 Heap 공간을 확보하는 과정에서 Swapping 까지 일으킬 가능성이 있어서가 아닐까 싶다!

아 그리고, Auto Heap Sizing 을 바로 사용할 것인가 하면 그것도 아닌데,
이유는 이 기능을 사용하다가 ES 가 멋대로 메모리를 열심히 사용하는 걸 OS 가 못견디고 ES 프로세스를 시스템 단에서 죽여버리는 순간
ES 는 자체 로그도 못남기고(syslog 쪽엔 남겠지만) 찍 소리도 못하고 kill 될 가능성이 있다.
이렇게 되면 ES 노드에 치명적인 문제가 발생하여 이후 복구하려고 노력해도 데이터 정합성 문제나 손상 등으로 클러스터에 못붙을 수도 있...(어후)

ES 가 이런 위험성도 충분히 고려해서 해당 기능을 내 놓았는지의 여부는 차후 테스트 등을 통해 살펴보면 좋을 것 같다.

### 참고
* https://www.elastic.co/guide/en/elasticsearch/reference/master/advanced-configuration.html
* https://www.elastic.co/guide/en/elasticsearch/reference/master/setup-configuration-memory.html
* https://www.elastic.co/guide/en/elasticsearch/reference/master/_memory_lock_check.html
* https://www.nakjunizm.com/2017/09/07/ElasticSearch_Heap_Memory/
* https://kyeoneee.tistory.com/58
