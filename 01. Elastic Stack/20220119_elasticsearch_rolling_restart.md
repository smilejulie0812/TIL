# Rolling Restart 수순서
Elasticsearch 관련 엔지니어로서 가장 하기 귀찮은 작업이 Rolling Restart 일 것이다 왜냐면 노드 수가(정확히는 샤드 수가) 많을수록 소요 시간이 길어지기 때문이지...

예전 TIL 에서 필요한 API 를 리스트화해둬서 중복 내용이 좀 있긴 한데, 내일 작업할 때 그대로 따라할 수 있을만큼의 간단한 수순서를 만들어두면 두고두고 잘 쓸 수 있을 것 같아서 이참에 간단히 만들어 둔다.

```json
### 1. 클러스터 레벨에서 Shard Allocation 하는 수 늘려주기

PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.node_concurrent_incoming_recoveries": 32,
    "cluster.routing.allocation.node_concurrent_outgoing_recoveries": 32,
    "cluster.routing.allocation.node_initial_primaries_recoveries": 32,
    "cluster.routing.allocation.cluster_concurrent_rebalance": 32
  }
}

### 2. 클러스터 인바운드/아웃바운드 트래픽 제한 수치 높여주기
PUT _cluster/settings
{
  "transient": {
    "indices.recovery.max_bytes_per_sec": "512mb"
  }
}

### 3. Shard Rebalancing 비활성화 : none 으로 조정
PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.rebalance.enable": "none",
    "cluster.routing.allocation.enable": "none"
  }
}

### 4. Fluch Synced : flush 한 후, 샤드마다 고유한 sync-id 를 발급해서 비교하게 한다
### -> 클러스터 recovery 나 재시작시 샤드 비교에 대한 효율성이 좋아짐
POST _flush/synced

### 5. 1대씩 설정 수정하고 재시작, Data Node 의 경우 Yellow 에서 Green 으로 바뀌는 것 확인하며 1대씩 진행
### -> 재시작 노드 순서는 Coordinate -> Master -> Data 순서대로 한다

### 클러스터 모니터링용 API
##### 클러스터 전체 헬스체크. v 옵션은 위의 머릿글을 함께 표시 
GET _cat/health?v
##### 클러스터 노드 체크
GET _cat/nodes?v
##### 대기 중인 태스크의 리스트를 출력
GET _cat/pending_tasks

### Shard Rebalancing 활성화 : all 으로 조정
PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.rebalance.enable": "all",
    "cluster.routing.allocation.enable": "all"
  }
}
```
## 참고
* https://www.elastic.co/guide/en/elasticsearch/reference/current/restart-cluster.html#restart-cluster-rolling
* https://www.popit.kr/%EC%97%98%EB%9D%BC%EC%8A%A4%ED%8B%B1%EC%84%9C%EC%B9%98es-%ED%81%B4%EB%9F%AC%EC%8A%A4%ED%84%B0-%EC%9E%AC%EC%8B%9C%EC%9E%91-%ED%98%B9%EC%9D%80-%EC%97%85%EA%B7%B8%EB%A0%88%EC%9D%B4%EB%93%9C-tip/
