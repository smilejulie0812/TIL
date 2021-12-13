## Elastic Stack 과 Kafka 를 사용해서 로그 데이터 파이프라인을 만들자
### 서론
저번에 Elasticsearch 노드 systemd 파일에 자동 restart 설정을 걸어둔 후로 정확한 원인 분석하는 게 번거로워진 이유로,  
해당 Elasticsearch 노드마다의 로그 중 이슈 관련 로그만 teams 로 받아보기로 했다.

### 구성
TIL 문서이므로, 대략적인 데이터 흐름을 적어본다.  
```
Elasticsearch 노드 - (Filebeat) --> Kafka --> Logstash --> Elasticsearch --> ElastAlert --> Teams
```

### 설정
진척된 상황까지만 기록해 두고, 남은 부분은 다른 날 TIL 문서에 남기든 이 문서에 추가하든 하도록 한다.

#### filebeat.yml
```
filebeat.config.inputs:
  enabled: true
  path: configs/*.yml
  reload.enabled: false

#============================= Filebeat modules ===============================

filebeat.config.modules:
  path: ${path.config}/modules.d/*.yml
  reload.enabled: false
  #reload.period: 10s

#---------------------------- kafka output -------------------------------------
output.kafka:
  enabled: true
  hosts: [""]
  topic: '%{[log_topic]}'
  partition.round_robin:
    reachable_only: false

  required_acks: 1
  compression: gzip
  max_message_bytes: 1000000
```

#### elasticlog.yml
```
- type: log
  paths:
    - /var/log/elasticsearch/*.log
  fields_under_root: true
  fields:
    log_topic: elasticlog
  include_lines: [ '\[WARN', '\[ERROR' ]
  multiline.type: pattern
  multiline.pattern: '^\[[0-9]{4}-[0-9]{2}-[0-9]{2}'
  multiline.negate: true
  multiline.match: after
```

#### kafka topic
```bash
### 토픽 생성
bin/kafka-topics.sh --create --zookeeper {IP 주소}:2181 --partitions {partition 수} --replication-factor {replication 수} --topic elasticlog
### 테스트할 때 오프셋값 확인 : 데이터 프로듀스&컨슘이 정상적으로 일어나는가
bin/kafka-consumer-groups.sh --bootstrap-server localshot:9092 --describe --group elasticlog
### LOG-END-OFFSET 값은 프로듀싱된 상태를, CURRENT-OFFSET 값은 컨슘해 간 상태를 확인할 수 있다.
### LAG 값은 LOG-END-OFFSET - CURRENT-OFFSET 이므로 0이면 프로듀싱된 메시지를 모두 컨슘해 갔다는 의미
```

#### logstash pipeline.conf
```
input {
    # text format
    kafka {
        bootstrap_servers   => "{IP 주소}:9092"
        topics              => "elasticlog"
        group_id            => "elasticlog"
        consumer_threads    => "5"  ### kafka 브로커가 5 대 구조이고 partition 수도 이에 맞춰 5 개로 설정
        type                => "elasticlog"
    }
}

###########################################################################################################

filter {
    if [type] == "elasticlog" {
        grok {
            match => {
                "message" => [ "\[%{TIMESTAMP_ISO8601:log-timestamp}\]\[%{GREEDYDATA:loglevel}\]\[%{DATA:component}\] \[%{DATA:hostname}\] %{GREEDYDATA:msg}" ]
            }
            timeout_millis => 1000
        }

        mutate { add_field => { "index-timestamp" => "%{@timestamp}" } }

        date {
            match => [ "log-timestamp", "ISO8601" ]
            timezone => "Asia/Seoul"
        }

        mutate { remove_field => [ "log-timestamp" ] }

        ruby {
            init => "require 'time'"
            code => "
                    event.set('msg', event.get('msg')[0..100]);     ### 100 자가 넘는 메시지는 이슈 발생의 원인이 될 수 있으므로 자른다
                    event.set('indexDate', Time.now.utc.getlocal.strftime('%Y.%m.%d'));
                    event.set('indexDateHour', Time.now.utc.getlocal.strftime('%Y.%m.%d.%H'));
                "
        }
    }
}

###########################################################################################################

output {
    if [type] == "elasticlog" {
        elasticsearch {
            hosts               => ["{IP주소}:9200"]
            index               => "elasticlog-%{indexDateHour}"
            document_type       => "elasticlog"
        }
    }
}
```

### 과제
* 파이프라인 필터 부분에서 grok 을 거친 message 필터가 자동으로 사라지고 새로 생성되지도 않는 현상이 발생.
* filebeat 에서 loglevel 설정하는 방법
* filebeat 의 multiline 설정이 제대로 먹는지 확인 필요
* 아래 내용 참고로 ES grok filter 수정
```
grok {
      match => ["message", "\[%{TIMESTAMP_ISO8601:timestamp}\]\[%{DATA:loglevel}%{SPACE}\]\[%{DATA:source}%{SPACE}\]%{SPACE}\[%{DATA:node}\]%{SPACE}\[%{DATA:index}\] %{NOTSPACE} \[%{DATA:updated-type}\]",
                "message", "\[%{TIMESTAMP_ISO8601:timestamp}\]\[%{DATA:loglevel}%{SPACE}\]\[%{DATA:source}%{SPACE}\]%{SPACE}\[%{DATA:node}\] (\[%{NOTSPACE:Index}\]\[%{NUMBER:shards}\])?%{GREEDYDATA}"
      ]
   }
참고) https://logz.io/blog/logstash-grok/
```

