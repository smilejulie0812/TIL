# Elasticsearch 의 설정 적용 순서
Elasticsearch 의 큰 특징 중 하나는 REST API 를 지원한다는 사실일 것이다. 실제로 클러스터 전체에 대한 설정을 변경할 때, 웬만한 설정은 클러스터 전체에 대한 API 로 적용시킬 수 있다(특히 Kibana 의 Dev Tools 에서 API 를 작성할 경우 자동 완성까지 해 주기 때문에 편리하다). 이 API 를 통한 설정 방법은 클러스터가 움직이고 있는 동안에 설정할 수 있으며, 동적 설정이 가능하기 때문에 클러스터를 멈출 필요가 없다.

지금까지 수많은 API 를 날리면서 작업을 해 왔는데, 머리 속에 제대로 각인되지 않았던 부분이 바로 ‘transient’ 와 ‘persistent’ 이다. 직역하면 ‘일시적인’ 과 ‘영구적인’ 인데, 무엇을 기준으로 일시적이고 영구적인 설정인지, 그리고 어떤 설정이 우선시되는지 등의 개괄적인 내용을 정리해 두면 좋을 것 같아 TIL 에 기재하기로 한다.

## transient vs persistent setting

무엇이 ‘일시적’이고 ‘영구적’인가? 그 기준은 바로 **클러스터의 재시작 여부**이다. 풀어쓰자면, **transient setting 의 경우 해당 클러스터가 재시작될 경우 설정이 날아가는(디폴트값으로 변하는) 설정**이다. 반면에 **persistent setting 은 해당 클러스터를 재시작하더라도 이전에 한 설정이 날아가지 않고 그대로 적용되는 설정**이다.

보통 영구적인 설정은 API 가 아닌 elasticsearch.yml 와 같은 설정 파일에 기재하는 경우가 많은데, Elasticsearch 의 경우 REST API 를 사용해서도 영구적인 설정을 적용할 수 있다는 말이다.

여기서 막간 상식. API 로 어떤 설정값을 기본값이 아닌 값으로 적용했다가 다시 기본값으로 돌리고 싶을 경우에는 어떻게 할까? DELETE 메소드로 해당 설정값을 지워야 하나? 이 땐 해당 설정에 대한 값을 null 로 지정하여 PUT/POST 함으로서 기본값 설정을 할 수 있다. 예를 들면 이런 식으로.

```json
PUT _cluster/settings
{
  "transient": {
    "indices.recovery.max_bytes_per_sec": null
  }
}
```

## Setting 적용 순서

Elasticsearch 에 설정을 넣는 방법은 크게 REST API 설정과 설정 파일(elasticsearch.yml) 을 사용하는 방법으로 나뉘고, REST API 설정은 또 다시 transient setting 과 persistent setting 으로 나뉜다. 다만 같은 설정이 중복되거나 할 경우, 어떤 순서로 설정이 적용될까?

1. **Transient Setting** : REST API 일시 설정(클러스터 재시작되면 기본값으로 되돌아감)
2. **Persistent Setting** : REST API 영구 설정(클러스터 재시작되어도 설정값 유지)
3. **elasticsearch.yml 파일 Setting** : 해당 노드에 일일이 설정 후 노드 재시작할 필요가 있음
4. **Default Setting** : 설정 기본값

elasticsearch.yml 에 설정값을 변경하는 것이 파일로 증거가 남기 때문에 편리할 수는 있다. 다만 이 경우에는 설정을 바꾼 노드를 재시작해야 할 필요가 있기 때문에, 클러스터 상태를 무너뜨리지 않고 설정을 변경하기 위해서는 REST API 를 통한 설정 적용이 유용하다(물론, 모든 설정을 REST API 로 설정할 수 있는 것은 아니므로 이 부분은 조사하면서 설정해야 할 것이다).

## 참고

- [https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html)
