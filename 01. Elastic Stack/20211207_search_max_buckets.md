## Aggregation의 종류와 search.max_buckets 값의 관계성
어제 관리 중인 Elasticsearch 노드 중 하나가 또 떨어졌는데(지긋지긋한 OutOfMemory...)  
이 노드에 deprecated 로그가 찍혀있길래 관련 문서 거슬러 거슬러 찾아본 기록을 남겨둔다.

### Aggregation(집계)
Elasticsearch 의 **Aggregation(집계)** 기능은 기존 데이터를 일정 기준에 따라 데이터를 그룹화하는 기능이다.  
(SQL GROUP BY 와 비슷한 기능이라고 생각하면 될 듯)  
하나의 응답에서 검색 결과를 반환함과 동시에 집계 결과 역시 함께 반환할 수 있다.

집계의 종류는 크게 세 가지로 나뉘는데,
* **Metric Aggregation** : 필드 값의 합계나 평균 등의 메트릭 값을 집계
* **Bucket Aggregation** : 필드의 value 나 범위 등, bin 이라 불리며, document 를 bucket 이란 개념으로 그룹화한 결과를 집계
* **Pipeline Aggregation** : document 나 field 대신 다른 aggregation 을 기준으로 집계

그 중에서 오늘의 주제는 **Bucket Aggregation** 이다.
공식 문서의 내용을 요약해서 적어두면,
- Bucket Aggregation 은 해당 document 의 bucket 을 만든다
   (필드의 메트릭값을 계산해서 출력하는 Metric Aggregation 의 방식과 다름)

- 각각의 Bucket 은 aggregation 타입에 따라 결정되는 기준에 포함되는 document 과 관련되는 개념
-> 즉, Bucket 이라는 것은 어떠한 기준에 대한 document 의 그룹? 세트? 라고 생각하면 될 듯

- Bucket aggregation 은 각 Bucket 에 포함된 document 의 count 값도 반환한다.

- Bucket aggregation 은 Metric aggregation 과 반대 개념

- Bucket aggregation 은 sub-aggregations 을 구성할 수 있고, 이 sub-aggregations 은 parent bucket aggregation 에서 생성된 bucket 에 의해 집계된다

- Bucket aggregation 은 각각의 다른 방법으로 버킷을 만들 수 있다(이를 bucketing strategy, 버킷링 전략을 사용하는 bucket aggregator 라고 한다).
  그리고 이 bucket aggregator 는 단일 bucket 을 정의할 수도, 복수 bucket 을 정의할 수도, 또한 aggregate(집계)하는 프로세스 중에 동적으로 bucket 을 생성할 수도 있다.

### search.max_buckets
이제 여기서 나오는 개념이 바로 **search.max_buckets** 이다.  
이 설정의 의미는, 1개의 response 에 대해 최대로 허용되는 bucket 의 수를 지정할 수 있다는 의미이며, 동적 클러스터 설정에 의해 제한된다.  
버전 6 기준으로 기본값이 -1 로 ‘사용 안함’ 이고, 생성되는 bucket 수가 10000 개를 넘어서면 ES 의 deprecation 로그에 경고 메시지를 출력한다.  

다만, 단일 aggregation 이 아닌 복합 aggregation 을 사용할 때에는 이 -1 설정의 움직임이 달라진다.  
이 때에는 Soft limit 수치를 Hard limit 수치 대신 인식해서 로그에 TooManyBucketsException 메시지를 출력할 것이다.  
이 Soft Limit 을 초과하여 위의 상황이 발생할 때에는 값을 10000 이하로 설정해 주어야 한다.  

버전 7 기준으로 기본값은 65,536 으로 설정되어 있다.  
단일 응답 기준으로 허용 가능한 최대 bucket 수이며, 이 값을 초과하면 오류 메시지가 반환된다.

## 설정 방법
클러스터 기동 중에 아래 API 로 쉽게 설정치를 변경할 수 있다.
```json
PUT _cluster/settings
{
  "persistent": {
    "search.max_buckets": 20000
  }
}
```

## bucket aggregation 과 성능 관계성
https://www.elastic.co/kr/blog/advanced-tuning-finding-and-fixing-slow-elasticsearch-queries
뭐 당연하겠지만... 고유한 필드를 집계할 때 힙 사용량이 증가한다고 한다.  
특히 힙 덤프 분석 중에 search, buckets, aggregation 등의 용어가 포함된 JAVA 객체가 힙을 많이 사용한다고...

## 참고
https://www.elastic.co/guide/kr/elasticsearch/reference/current/gs-executing-aggregations.html
https://coding-start.tistory.com/291
https://www.elastic.co/guide/en/elasticsearch/reference/6.8/search-aggregations-bucket.html
https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket.html
https://www.elastic.co/guide/en/elasticsearch/reference/current/search-settings.html#search-settings-max-buckets
