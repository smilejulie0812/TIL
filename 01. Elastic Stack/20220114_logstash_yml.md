## logstash.yml 를 정리해보았다
시스템 재구축을 위해 기존 시스템의 logstash 설정파일을 정리하다가 새로 알게 된 개념도 있어서 겸사겸사 정리해둔다. 일단 TIL 페이지에 간단하게 정리해 두었다가 개념 정리 더 해서 블로그에 승격시켜야지.

- logstash.yml

```bash
# pipeline 돌릴 때, input 단계에서, filter & output 까지 한 번에 묶어서 처리하기 위해, 스레드에서 얼마나 많은 (로그)이벤트를 모을 것인가? 이걸 ‘이벤트 배치’라고 하고, 그 이벤트 배치의 사이즈를 의미한다. 기본값 125
pipeline.batch.size: 2048
# 하나의 이벤트 배치가 아직 최대 사이즈까지 차오르지 않았을 때, 이걸 pipeline worker 에게 보내서 처리하기 위해 추가 이벤트를 기다리기 위한 최대 시간. 기본값 50 밀리초
pipeline.batch.delay: 100

# pipeline 의 input 과 filter 사이에 존재하는 Message Queue 를 사용할 것인가에 대한 설정
# 기본값은 memory 로, Message Queue 를 메모리에 저장하는 방식.
# 해당 값을 persisted 로 설정하면 input 과 filter 사이에 ‘Persistent Queue’ 를 두어, 
# 장애 발생시 이 PQ 에 저장된 데이터들은 메모리가 아닌 디스크의 지정된 경로에 저장되기 때문에 서버가 재시작되면 순차적으로 처리할 수 있다
# 데이터 유실을 방지하기 위해서는 사용하되, 빠른 처리를 원할 경우에는 굳이 사용하지 않아도 될듯?
# input 받은 모든 데이터를 정확하게 처리하기 위해서는 persistent queue 를 설정해서, 
# input 된 데이터는 logstash 서버 재시작 후에도 처리할 수 있도록 하는 것이 안정적이다.
queue.type: memory
# 아래 설정은 queue type 을 persistent 로 설정했을 때 사용하는 옵션들
queue.page_capacity: 128mb
queue.max_events: 0
queue.max_bytes: 256mb
queue.checkpoint.acks: 1024
queue.checkpoint.writes: 1024
queue.checkpoint.interval: 1000

path.config: /etc/logstash/conf.d
path.logs: /var/log/logstash

# logstash 솔루션의 로그 파일(logstash-plain.log)를 debug 레벨까지 기록하면 로그 데이터가 대량으로 발생하기 때문에 error 로그만 따로 기록
log.level: error

# monitoring 이나 management 를 위한 elasticsearch 주소는 전용 coordinate node 를 사용하는 것이 제일 좋을 것으로 판단
xpack.monitoring.elasticsearch.url: "http://<Elasticsearch Node>:9200"
```

### 과제

- Logstash 의 Memory Message Queue 와 Disk 를 사용하는 Persistent Queue 비교

### 참고

- [https://www.elastic.co/guide/en/logstash/7.16/logstash-settings-file.html](https://www.elastic.co/guide/en/logstash/7.16/logstash-settings-file.html)
- [https://www.elastic.co/guide/en/logstash/current/memory-queue.html](https://www.elastic.co/guide/en/logstash/current/memory-queue.html)
- [https://www.elastic.co/guide/en/logstash/current/persistent-queues.html](https://www.elastic.co/guide/en/logstash/current/persistent-queues.html)
- [https://injekim97.tistory.com/240](https://injekim97.tistory.com/240)
