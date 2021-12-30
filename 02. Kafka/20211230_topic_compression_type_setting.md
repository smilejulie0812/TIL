# Topic Compression Type Setting
새로운 토픽을 생성할 때 신경썼던 건 항상 파티션과 레플리카 수 정도였는데, 이번에 토픽 생성할 때 동시에 압축 타입을 지정해주길 요청하는 안건이 들어와서 간단하게 메모해 둔다.

(Kafka 의 압축 타입은 Topic 또는 Broker 단계에서 지정해 줄 수 있다.)

## Topic compression.type

해당 토픽의 압축 형태를 의미하며, 이 토픽에 보관되는 메시지를 압축함으로서 디스크 및 네트워크 리소스를 절약할 수 있다.

다만 압축과 해동 작업을 진행하는 데 CPU 를 사용한다는 점이 있지만, 이 메시지 압축이 프로듀서/컨슈머의 처리량을 높여준다고 하니, 운영 환경에 맞는 압축률을 고려해서 compression.type 을 정하는 것이 효율적인 브로커 운영의 방법이 될 것이다.

## Compression Type

- 기본값 : none ( 압축하지 않음 )
- producer : 프로듀서에서 사용하는 압축 방법을 그대로 사용한다

Kafka 에서 제공하는 compression type 은 다음의 네 가지가 있고, 각각의 특징은 표로 그릴 수 있다.

| 압축 타입 | 압축률 | CPU 사용량 | 압축 속도 | 네트워크 대역폭 사용량 |
| --- | --- | --- | --- | --- |
| gzip | 가장 높음 | 가장 높음 | 가장 느림 | 가장 낮음 |
| snappy | 중간 | 중간 | 중간 | 중간 |
| lz4 | 낮음 | 가장 낮음 | 가장 빠름 | 가장 높음 |
| zstd | 중간 | 중간 | 중간 | 중간 |
- snappy : 가장 중간이라 활용성의 균형이 잘 맞는다
- zstd : snappy 보다 조금 더 많은 CPU 를 사용하되 좀 더 높은 압축률을 제공한다
    - Kafka 2.1.0 이후 버전에서 사용 가능한 타입

## compression.type setting command

토픽을 신규 생성할 때 config 추가하는 방식으로 설정해줄 수 있다.

```bash
### 압축 타입을 zstd 로 설정할 경우
<Kafka 경로>/bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor <레플리카 개수> --partitions <파티션 개수> --config compression.type=zstd --topic <토픽명>
```

## 과제

 linger.ms 및 batch.size 값의 설정 가치 조사

## 참고

- [https://getto215.github.io/kafka-architecture-2/](https://getto215.github.io/kafka-architecture-2/)
- [https://developer.ibm.com/articles/benefits-compression-kafka-messaging/](https://developer.ibm.com/articles/benefits-compression-kafka-messaging/)
- [http://happinessoncode.com/2019/01/18/kafka-compression-ratio/](http://happinessoncode.com/2019/01/18/kafka-compression-ratio/)
