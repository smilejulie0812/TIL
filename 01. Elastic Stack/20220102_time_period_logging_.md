# Time Period Logging Settings

Elasticsearch 는 log4j2 로 로깅한다는 사실은 저번 취약점 관련으로 하도 여기저기 포스팅해서 입이 아플 지경이다.

예전에 Elasticsearch 나 Logstash 의 로그 로테이션 설정을 할 때, TimeBasedTriggeringPolicy 는 로테이션 파일을 자동으로 삭제하는 기능이 log4j2 자체에 없다는 글을 어디선가 읽어서 아... 따로 logrotation 설정 넣어야하나 고민했는데, 등잔 밑이 어둡다고 충분히 로깅 가능한 기능을 Elasticsearch/Logstash 내 log4j2.properties 파일에서 제공해 주고 있었다(~~stackoverflow 도 좋지만 공식 페이지부터 찾자~~)

이번에 실적용하면서 사용하고 있는 설정을 아래에 간단히 기재해 둔다.

### Elasticsearch 의 경우

- <path>/elasticsearch/config/log4j2.properties

```bash
## 정책은 DefaultRolloverStrategy 를 사용
appender.rolling.strategy.type = DefaultRolloverStrategy 
### DefaultRolloverStrategy 정책 타입을 사용해서 시간 베이스의 로그를 삭제하기 위해서는 Delete 액션을 필수로 넣어줘야 한다
appender.rolling.strategy.action.type = Delete
### delete 액션 설정을 적용할 폴더명. 아래와 같이 작성할 경우, elasticsearch.yml 에서 path.logs 로 지정한 디렉토리를 바라본다 
appender.rolling.strategy.action.basepath = ${sys:es.logs.base_path} 
### Rollover 를 실행할 때의 조건 : 만약 파일 이름이,
appender.rolling.strategy.action.condition.type = IfFileName 
### 위의 조건에 대한 상세 내용 : 'elasticsearch.yml 에서 cluster.name 로 지정한 이름-*' 이라면
appender.rolling.strategy.action.condition.glob = ${sys:es.logs.cluster_name}-* 
### 첫 번째 조건에 중첩되는 조건 적용 : 마지막으로 수정되는 것이,
appender.rolling.strategy.action.condition.nested_condition.type = IfLastModified 
### 위의 조건에 대한 날짜 : 7일 이내의 것만 남기고 나머지는 삭제
appender.rolling.strategy.action.condition.nested_condition.age = 7D
```

### Logstash 의 경우

- <path>/logstash/config/log4j2.properties

```bash
appender.rolling.strategy.type = DefaultRolloverStrategy
appender.rolling.strategy.action.type = Delete
appender.rolling.strategy.action.basepath = ${sys:ls.logs}
appender.rolling.strategy.action.condition.type = IfFileName
appender.rolling.strategy.action.condition.glob = logstash-${sys:ls.log.format}-*
appender.rolling.strategy.action.condition.nested_condition.type = IfLastModified
appender.rolling.strategy.action.condition.nested_condition.age = 7D
```

### 참고

- [https://www.elastic.co/guide/en/elasticsearch/reference/8.0/logging.html](https://www.elastic.co/guide/en/elasticsearch/reference/8.0/logging.html)
