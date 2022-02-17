# Dangling Index

~~정식 명칭은 저렇지만 개인적으로는 좀비 인덱스라고 말하고 싶다~~

서버 상면 이동 프로젝트의 일환으로 기존에 운영되던 Elasticsearch 클러스터의 서버를 대대적으로 교체하는 작업이 있었다. 다른 클러스터 교체 작업과 다른 점이 있었다면 Coordinating Node 뿐 아니라 Master 및 Data Node 로 사용 중인 서버의 교체가 필요했던 것.

해당 작업을 위해 아래와 같은 순서로 교체 작업을 실행했다.

1. 신규 서버에 기존 클러스터와 동일한 구조 및 버전으로 Elasticsearch 신규 구성
2. 기존 클러스터에 1 의 노드를 붙여 사용
3. 혹시 모를 Data Node 의 디스크 부족 현상을 방지하기 위해 기존 Coordinating 전용 Node 로 쓰던 서버에 Data Node Role 을 부여 ~~대 삽질의 서막~~
4. 기존 노드 Shutdown 및 3 의 Data Node Role 회수

당시 작업은 매우 순조로웠고, 데이터 유실이나 클러스터 Red Status 이슈 없이 스무스하게 끝났다.

그런데!

이후로 별도 Batch 작업에 의해 삭제되었던 인덱스가 Red 상태로 살아나는 기현상이 벌어지기 시작한 것이다. 덕분에 클러스터 상태도 Red 로 떨어져서 모니터링 알람은 아비규환. 빈 인덱스로 살아난다면 차라리 Green 상태이기라도 할 법 한데, Kibana UI 상에서는 모니터링에 모습을 보이지도 않는 주제에 API 날려서 확인하면 존재하고. API 로 삭제하면 사라지고 클러스터도 곧장 Green 화 되지만, 이런 현상이 하루 몇 번씩이나 일어나면 곤란하다.

특히 주목할 것은, 위의 서버 교체 작업이 있었던 시간대의 인덱스만 좀비화되어 살아난다는 것이다. 즉 교체할 때 어떤 작업이 원인이 되어 해당 이슈가 발생했다는 건데... 해당 클러스터는 매우 오래된 레거시 시스템인지라 ILM 을 Elasticsearch 자체 서비스도, 심지어 Curator 도 아닌 Jenkins 에서 스크립트 배치를 돌리는 상황이었으므로 ES 와는 상관이 없었다. 신규 서버 Disk 에러인가 싶어서 아예 새로 서버를 파서 바꾸는 작업까지 했었지만 상황은 바뀌지 않았고... 그 와중에 알아낸 개념이 바로 **Dangling Index** 이다.

## Dangling Index 가 뭔데?

Elastic 공식 페이지에도 굉장히 짧은 설명만 적혀 있어서 번역해 보기로 한다.

> 한 노드가 어떤 Elasticsearch 클러스터에 참가했는데 그 노드 내부에 있는 Data Directory 에는 저장되어 있는 샤드 정보가 클러스터에 존재하지 않으면, 그 샤드는 Dangling Index 에 속하는샤드라고 말할 수 있다. 이러한 Dangling Index 는 전용 API 를 사용해서 리스트화 및 삽입, 삭제할 수 있다.
> 

<aside>
📌 Dangling Index 관련 API 로 얻어낸 결과는, 그 Dangling Index 가 클러스터에 존재하던 시절 가장 최신 상태의 정보였는지 보장할 수는 없다.

</aside>

즉, **클러스터에는 없는데 어떤 이유로 데이터 디렉토리에 일부 샤드 형태로 남아있는 인덱스를 Dangling Index** 라고 한다.

이쯤되면 무엇이 원인이어서 이 Dangling Index 가 만들어졌는지 짐작이 간다. 위의 서버 교체 작업 중 Coordinating Node 전용 노드에 Data Role 을 부여해서 임시로 사용하던 과정에서 해당 서버의 Data Directory 에 샤드 정보가 저장되었는데, 이후에 그 서버의 Data Role 을 회수하면서 디렉토리에 남은 정보를 삭제하지 않은 탓에, 불완전한 샤드 정보가 Dangling Index 로서 계속해서 올라오게 된 것. 그래서 해당 서버에 남아있던 Data Directory 의 정보를 말끔히 삭제해주자 거짓말처럼 이슈를 해결할 수 있었다.

### 참고

- [https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-gateway.html#dangling-indices](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-gateway.html#dangling-indices)
- [https://www.elastic.co/guide/en/elasticsearch/reference/master/indices.html#dangling-indices-api](https://www.elastic.co/guide/en/elasticsearch/reference/master/indices.html#dangling-indices-api)
