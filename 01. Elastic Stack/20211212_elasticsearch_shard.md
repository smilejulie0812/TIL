# 샤드 설계
## 서론

어떤 ES 클러스터를 설계할 때, 예상되는 데이터 유입량 정보로 가장 효율적인 설계를 하려면 어떻게 해야 할까?

물론 노드의 구성, 인덱스에 대한 설계도 중요하지만, 우선은 가장 작은 단위인 샤드를 설계하는 단계에서부터 시작해보자.

샤드를 어떤 식으로 설계할 것인가를 대강 잡고 나면 그에 맞는 노드와 인덱스 구성이 보일테니까.

## 샤드의 크기는 성능에 중요한 영향을 끼친다

1개의 샤드에 1개의 스레드(단일 스레드)가 하나의 쿼리를 실행한다.

하나의 샤드에 여러 쿼리를 날리거나 집계(Aggregation)를 실행하거나, 여러 샤드를 동시에 처리할 수도 있다.

즉, 쿼리의 응답 시간은 샤드의 크기에 따라 달라질 수 있다.

샤드가 작은 사이즈로 여러개라면? 갹 샤드마다의 속도는 빠르지만, 많은 작업을 큐에 넣고 순서대로 처리해야 함!

→ 큰 사이즈의 몇 개 안되는 샤드로 처리하는 것보다 반드시 빠르다고 단정지을 수는 없다

→ 즉, 1개의 샤드가 권장하는 최대 크기를 넘지만 않는다면, 큰 크기의 적은 샤드가 훨씬 효율적일 것

→ 제일 좋은 방법은 실제 데이터 및 쿼리로 벤치마크 테스트를 진행하는 것이다! 

## Primary Shard 는 변경할 수 없다

샤드 설계가 중요한 이유는, 어떤 인덱스를 처음 생성할 때 설정한 Primary Shard 개수는 바꿀 수 없기 때문일 것이다. (Primary Shard 개수를 바꾸기 위해서는 reindex 하는 수밖에 없다. Replica 는 변경 가능)

샤드의 수는 너무 많아도, 하나의 샤드가 너무 커도 문제가 될 수 있다.

- 샤드 수가 너무 많을 때 : 샤드를 운영하는 데에도 CPU, 메모리가 소모되기 때문에 비효율적
- 샤드 수가 너무 클 때 : 장애시 복구하기가 힘들다

## 노드와의 관계성

ES 7 버전에서는 노드 당 샤드의 개수를 1000 개로 제한하고 있으며, 변경하고 싶을 땐 옵션값을 바꿔주면 된다.

설정 가능한 옵션은 이거 : cluster.max_shards_per_node

## Best Practices

Elastic 사에서 제안하는 샤드 설계에서 고려할 점은 아래와 같다.

- 도큐먼트가 아니라 인덱스를 지워라
- 시간 기반의 데이터는 data stream 과 ILM 을 사용해라
- 샤드의 크기를 10GB ~ 50GB 사이로 설정해라
- 샤드의 개수를 Heap Memory 1GB 당 20개 이하로 설정해라
- 노드 핫스팟(특정 노드에 샤드가 특히 몰리는 경우)를 피해라
- 불필요하게 매핑된 필드를 줄여라
- (이미 샤딩이 완료된 경우) 클러스터의 샤드 count 를 줄여라

## 디스크 관점에서

하나의 샤드 당 권장하는 사이즈는 10 - 50GB (Elastic Korea 에서는 10 - 30GB 추천) 이다.

계산법) 데이터 사이즈가 60GB, 샤드 하나의 크기를 30GB 로 원한다면,

( 60 * 1.1 ) / 30 = 2개 ### 1.1 을 곱하는 이유는, 인덱스 자체의 크기가 10% 정도 차지하기 때문임

(소스 데이터 + 늘어날 공간의 예상치) * ( 1 + 인덱싱 오버헤드 ) / 원하는 샤드의 크기 = 샤드 수

→ 다만, 데이터가 늘어날 것을 예상해서 샤드 수를 늘려두는 것보다는, 처음에 적당한 개수로 사용하다가 추후 늘어날 때 리인덱스하는 것이 효율적이라고 한다.

## 메모리 관점에서

1개의 노드에서 Java Heap Size 1GB 당 샤드의 개수를 20개 이하로 설정해 준다.

예를 들어 한 데이터 노드의 메모리가 8GB 라고 한다면,

일반적으로 ES 에 지정해주는 Heap Size 는 절반인 4GB 를 할당해주고,

4GB 의 힙 사이즈에 최대로 할당될 수 있는 샤드의 수는 4GB * 20 개 = 80 개이다.

? 샤드 수가 80개 정도일 때 권장되는 샤드 사이즈는 5GiB 라고?

## AWS 에서 인스턴스 타입을 정할 때?

이건 노드 설계까지 확장해서 나아가는 건데, ES 를 AWS 의 EC2 에 올릴 경우(OpenSearch 가 아니라?) 어떻게 정해야 하느냐 한다면,

스토리지 100GB 당 최소 2 vCPU & 8GiB 메모리 스펙인 EC2 를 사용하길 권장한다고 한다(이 정도면 m5.large 가 해당)

## 샤드 크기를 관리하는 방법

- 시간 기반의 인덱스 : timestamp 로 관리된느 인덱스라면 시간 혹은 일별로 인덱스를 쪼개서 ILM 등으로 관리하면 편리하게 샤드 관리가 가능하다
- 다만, 시간에 따라 인덱싱량이 빈번하게 변화하는 경우라면 균일한 샤드 크기 관리를 위해 아래의 API 를 사용할 수 있다.
- Rollover 인덱스 API : 클러스터가 저장한느 도큐먼트와 인덱스의 크기, 저장할 수 있는 도큐먼트의 최대 기간을 지정할 수 있다. 여기서 지정한 제약 조건을 넘어서면 다운타임 없이 신규 인덱스를 생성해준다.

→ 시간 기반 & 시간이 지나가면서 데이터의 볼륨이 불규칙한 불변 데이터를 다룬다면 적합

- Shrink 인덱스 API : 기존의 인덱스를 더 적은 Primary Shard 를 가진 신규 인덱스로 변경해준다.

→ 특정 기간을 저장하는 인덱스를 노드에 분산하고 싶다면, 인덱싱이 이뤄지지 않는 시점에 이 API 로 프라이머리 샤드를 줄일 수 있다
## 참고

- [https://www.elastic.co/guide/en/elasticsearch/reference/7.16/size-your-shards.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.16/size-your-shards.html)
- [https://docs.aws.amazon.com/ko_kr/opensearch-service/latest/developerguide/sizing-domains.html#bp-sharding](https://docs.aws.amazon.com/ko_kr/opensearch-service/latest/developerguide/sizing-domains.html#bp-sharding)
- [https://ohgyun.com/795](https://ohgyun.com/795)
- [https://www.elastic.co/kr/blog/how-many-shards-should-i-have-in-my-elasticsearch-cluster](https://www.elastic.co/kr/blog/how-many-shards-should-i-have-in-my-elasticsearch-cluster)
- [https://www.elastic.co/guide/en/elasticsearch/reference/7.16/modules-cluster.html#shard-allocation-awareness](https://www.elastic.co/guide/en/elasticsearch/reference/7.16/modules-cluster.html#shard-allocation-awareness)
