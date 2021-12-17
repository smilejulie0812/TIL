# Elasticsearch Monitoring Tips
Elastic Stack 의 Rolling Upgrade 를 진행하면서 사용해 본 각종 작은 팁들을 한데 모아 정리해 보는 공간

## Shard Allocation 조정

재시작할 때 샤드 할당 설정이 자동으로 되어있을 때, 재시작된 노드 내 샤드를 다시 메모리로 올리는 작업이 상대적으로 느려질 수 있기 때문에 다음과 같이 샤드 할당 설정을 일시적으로 비활성화시킨다.

```json
// Shard Allocation 비활성화
PUT _cluster/settings
{
  "persistent": {
		"cluster.routing.allocation.enable": "none"
}
// Shard Allocation 활성화(기본값으로 전환)
PUT _cluster/settings
{
  "persistent": {
		"cluster.routing.allocation.enable": "all"
}
```

### 클러스터 레벨에서 Shard Allocation 설정

하나의 노드에서 동시에 복구하는 샤드의 개수를 조정해서 좀 더 빠르게 할당할 수 있게 한다.

```json
PUT _cluster/settings
{
  "persistent": {
		"cluster.routing.allocation.node_concurrent_incoming_recoveries": 32,
		// incoming recovery 는 rebalance 가 아니라면 대부분 replica 샤드
		"cluster.routing.allocation.node_concurrent_outgoing_recoveries": 32,
		// outcoming recovery 는 rebalance 가 아니라면 대부분 primary 샤드
		"cluster.routing.allocation.node_initial_primaries_recoveries": 32,
		// 한 노드 내에서 초기 기본 복구가 동시에 수행되는 샤드의 수
		"cluster.routing.allocation.cluster_concurrent_rebalance": 32
		// 클러스터 내에서 동시에 rebalance(재조정)할 샤드 수
		""
}
```

## Shard Rebalancing 조정

노드 간 샤드 수/사이즈를 리밸런싱하는 작업 역시 재시작된 노드 내의 샤드가 메모리로 올라오는 데 사용될 리소스를 함께 사용하게 될 것이므로(특히 리밸런싱의 경우엔 노드가 재시작되면서 리밸런싱 대상에 참가하는 데 드는 리소스도 상당할 것이다), 다음과 같이 샤드 리밸런싱 설정을 일시적으로 비활성화시킨다.

```json
// Shard Rebalancing 비활성화
PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.rebalance.enable": "none"
  }
}
// Shard Rebalancing 활성화
PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.rebalance.enable": "all"
  }
}
```

## max_bytes_per_sec

각 노드마다 인바운드/아웃바운드 복구 트래픽을 제한하는 값으로, 노드마다 각각 다른 값을 지정해 줄 수 있다.

이 수치가 너무 낮으면, 클러스터 내의 여러 노드가 동시에 샤드를 복구할 때 상한치를 초과해서 복구 시간이 장기간 소요될 수 있다.

반면 수치가 너무 높으면, 지속적인 복구로 인해 리소스가 낭비되어서 클러스터가 불안정해질 위험이 있다.

기본값은 40mb/s 인데, 이 값은 총 메모리 4GB 를 기준으로 잡아둔 수치이므로,

Rolling Upgrade 같이 지속적인 복구 작업이 필요할 때에는 일시적으로 수치를 높여두는 것도 하나의 방법이다.

```json
PUT _cluster/settings
{
  "transient": {
		"indices.recovery.max_bytes_per_sec": "512mb"
}
```

## 유용한 모니터링 API

Elasticsearch 는 노드 한 대 씩 Yellow → Green 으로 전환되는 것을 확인하면서 업그레이드할 필요가 있는데, 이 때 나름 유용했던 API 를 간략히 적어둔다.

```json
// 클러스터 전체 헬스체크. v 옵션은 위의 머릿글을 함께 표시 
GET _cat/health?v
// 대기 중인 태스크의 리스트를 출력
GET _cat/pending_tasks

// Kibana -> Monitoring -> Overview -> Shard Activity
// 각 인덱스마다 샤드의 이동 현황을 한 눈에 볼 수 있다
```

## 플러그인 버전 관리

일반적으로 솔루션 간(ES - Kibana - Logstash)의 버전은 가장 마지막단 마이너 버전은 달라도 중간 버전까지만 맞으면 호환성이 맞아서 기동 및 클러스터화에 큰 문제가 없다.

(ES 6.8.4 → 6.8.21 로 업그레이드해도 문제 없이 클러스터에 붙는다는 의미)

다만, 솔루션 내에서 사용하는 플러그인은 모든 버전이 정확히 일치해야만 정상적으로 움직인다.

플러그인 설치 방법은 아래에...

```bash
### 외부 네트워크 연결된 경우
sudo bin/elasticsearch-plugin install <플러그인 이름>

### 외부 네트워크가 연결되지 않은 경우
##### 각 플러그인을 직접 다운로드받아(Elastic URL 참고) 해당 노드의 서버에 배치시킨다.
sudo bin/elasticsearch-plugin install file:<///path/to/plugin.zip>
```

## 참고

- [https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-cluster.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-cluster.html)
- [https://www.elastic.co/guide/en/elasticsearch/reference/current/recovery.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/recovery.html)
- [https://logz.io/blog/elasticsearch-cheat-sheet/](https://logz.io/blog/elasticsearch-cheat-sheet/)
- [https://lasel.kr/archives/495](https://lasel.kr/archives/495)
- [https://dol9.tistory.com/276](https://dol9.tistory.com/276)
- [https://www.elastic.co/guide/en/elasticsearch/plugins/current/installation.html](https://www.elastic.co/guide/en/elasticsearch/plugins/current/installation.html)
- [https://www.elastic.co/guide/en/elasticsearch/plugins/current/plugin-management-custom-url.html](https://www.elastic.co/guide/en/elasticsearch/plugins/current/plugin-management-custom-url.html)
