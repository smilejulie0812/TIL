Elasticsearch 클러스터가 디스크 사이즈 이슈로(데이터 노드에 따라 디스크 크기가 제각각임) 장기간 장애 상태이다.
특정 노드의 디스크가 85% 이상이며, 해당 노드에 할당된 샤드가 모두 primary node 0 번인 상황.
일시적인 문제 해결을 위해 해당 노드에 할당된 샤드를 다른 노드로 이동시켰으나, 인덱스가 새로 생성될 때 해당 노드로 다시 할당되는 통에 결국 제자리로 돌아오게 되었다.
장기적으로는 시스템의 재구축을 시행할 예정이나, 그 전까지 장애가 발생하는 것을 막기 위해 해당 노드에 일정 디스크 용량을 사용하게 되면 allocation 을 막도록 설정하게 되었다.
이 설정을 위해 필요한 옵션을 아래에 기재한다.

설정 방법
두 가지가 있다. elasticsearch.yml 에 옵션을 기재하거나, run 중인 클러스터에 API 로 날리거나.
설정 파일에 옵션 기재하는 것은 간단하므로 클러스터에 날릴 API 의 예를 적어둔다.

PUT /_cluster/settings
{
  "persistent" : {
    "cluster.routing.allocation.disk.threshold_enabled" = "true"                 ### Elasticsearch 의 디스크 할당자(Disk Allocation Decider) 사용을 허용. default 가 true 이므로 설정 불필요
    "cluster.routing.allocation.disk.watermark.low" = "70%"                 ### 디스크를 70% 이상 사용하는 노드에는 샤드를 추가적으로 할당하지 않는다. default = 85%
    "cluster.routing.allocation.disk.watermark.high" = "80%"                    ### 디스크를 80% 이상 사용하는 노드에 있는 샤드를 다른 노드에 재할당한다. default = 90%
    "cluster.routing.allocation.disk.watermark.flood_stage" = "95%"                 ### 디스크를 95% 이상 사용하는 노드의 경우, 해당 노드에 하나 이상 샤드가 할당된 인덱스를 read-only 로 block 처리한다. default 가 95% 이므로 설정 불필요
  }
}

cluster.routing.allocation.disk.threshold_enabled
디스크 할당자(디스크 기반으로 shard allocation 하는 기능)을 사용할 것인지 아닌지를 설정. 기본값은 true 이므로 굳이 설정할 필요 없다
cluster.routing.allocation.disk.watermark.low
노드가 설정 수치 이상으로 사용될 경우, 그 노드에는 샤드를 추가적으로 할당하지 않는다. 기본값 85%
cluster.routing.allocation.disk.watermark.high
노드가 설정 수치 이상으로 사용될 경우, 그 노드에 있는 샤드를 다른 노드에 재할당한다. 기본값 90%
cluster.routing.allocation.disk.watermark.flood_stage
노드가 설정 수치 이상으로 사용될 경우, 그 노드에 하나 이상 할당된 인덱스에 대해 read-only 로 전환, 즉 block 처리한다. 기본값은 95%
→ 이 상태에 빠질 경우, 이 컨플처럼 수동으로 block 을 풀어줄 필요가 있다.
cluster.info.update.interval
노드의 디스크 사용량을 체크하는 주기. 기본값은 30s(30초)
cluster.routing.allocation.disk.include_relocations
노드의 디스크 사용량을 체크할 때, 그 노드로 relocate 중인 샤드까지 고려할 것인가 아닌가를 설정

참고
https://www.elastic.co/guide/en/elasticsearch/reference/6.8/disk-allocator.html
https://qummi.tistory.com/26
https://medium.com/@zoo5252/elastic-search-%EB%94%94%EC%8A%A4%ED%81%AC-%EC%9A%A9%EB%9F%89-%EC%A0%9C%ED%95%9C-55e58e51ce85
