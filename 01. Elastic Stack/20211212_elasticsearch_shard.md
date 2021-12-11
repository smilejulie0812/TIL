## 샤드 설계
이 부분은 주말에 정리하고 공부해서 작성하고 날짜도 바꾸어 두는 걸로. 일단 메모 레벨로.

cluster.max_shards_per_node 설정으로 알아보는 샤드 설계

Aws 중심
https://docs.aws.amazon.com/ko_kr/opensearch-service/latest/developerguide/sizing-domains.html
https://ohgyun.com/795

Shard 설계
https://www.elastic.co/guide/en/elasticsearch/reference/7.16/size-your-shards.html
https://www.elastic.co/guide/en/elasticsearch/reference/7.16/modules-cluster.html#shard-allocation-awareness
https://www.elastic.co/guide/en/elasticsearch/reference/7.16/modules-cluster.html#shard-allocation-awareness

—

샤드 1개의 크기 : 10~30GB 가 적당함
->

JVM Heap Size 1GB 당 샤드의 개수가 20개를 넘지 않도록 한다
