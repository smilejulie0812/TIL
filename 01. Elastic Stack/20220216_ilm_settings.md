# Elastic ILM Settings
어떤 문제도 없이 잘만 설정되던 ILM 설정이 갑자기 에러를 내기 시작해서 꽤 많이 삽질했었는데, 의외로 정말 아무 것도 아닌 곳에서 헤매고 있었다는 사실을 깨닫고 작성하는 TIL.

이 포스팅에서는 Auto Rollover 설정이 들어가지 않는 기본적인 ILM 설정과 Auto Rollover 설정을 함께 지정하는 ILM 설정을 함께 기재하도록 한다.

## Basic ILM Settings

Logstash 에서 Elasticsearch Output Plugin 을 사용하여 Elasticsearch 에 데이터를 적재할 때, `index` 라는 옵션을 사용하면 (보통은) 시간별로 인덱스를 생성할 수 있다.

### Logstash Pipeline

```json
input { ... }
filter { ... }
output {
  elasticsearch {
    hosts => ["<Node#01>:9200",...,"<Node#NN>:9200"]
    index => "test-index-%{+YYYY.MM.dd}"
  }
}
```

위와 같이 pipeline 을 작성했다는 것은, 즉 하루별로 인덱스를 rollover 하겠다는 의미이다.

이런 설정에 대해서 ILM 을 설정하기 위해서는 아래와 같이 가장 기본적인 설정을 부여하면 된다.

### ILM Policy

```json
PUT /_ilm/policy/test-ilm-policy
{
##### HOT 옵션은 필수 설정이므로 그대로 둠
  "policy": {
    "phases": {
        "hot" : {
          "min_age" : "0ms",
          "actions" : { }
        },
##### 7일이 지난 인덱스는 삭제
      "delete": {
        "min_age" : "7d",
        "actions": {
          "delete" : { }
        }
      }
    }
  }
}
```

### Index Template - Only Settings

```json
{
  "index": {
    "lifecycle": {
      "name": "test-ilm-policy"
    },
    "number_of_shards": "3",
    "number_of_replicas": "1"
  }
}
```

## 왜 삽질을 했는가

ILM 이나 Index Template 설정은 Kibana Dev Tools 에서 API 를 날려서 적용시킬 수 있지만, 최근엔 GUI 환경에서도 손쉽게 설정할 수 있다.

근데 이게 폐해가 될 수 있다. 왜냐하면, ILM 을 설정하는 GUI 페이지에서 애초에 Auto Rollover 설정을 유효화시켜두었기 때문... 위에서 언급한 바와 같이 Logstash 에서 이미 Rollover 되도록 설정이 되어있는데 Auto Rollover 설정이 들어간 ILM Policy 를 지정하면, Logstash 의 `index` 옵션보다 `Auto Rollover` 옵션이 우선적으로 적용되기 때문에 설정 간 충돌이 일어난다. 어떻게든 Auto Rollover 를 사용하기 위해 필요한 추가 옵션을 설정하라는 에러 메시지. 그래, 바로 이런...

> illegal_argument_exception: setting [index.lifecycle.rollover_alias] for index [<인덱스명>] is empty or not defined
> 

~~하도 봐서 꼴도 보기 싫음...~~

이 삽질을 하지 않으려면 아주 간단하다. ILM 설정할 때 아래와 같이 Auto Rollover 기능을 무효화하기만 하면 되는 것.

[]()

## Auto Rollover ILM Settings

삽질이 길고 지지부진하기는 했지만, 항상 그렇듯 삽질은 여러 배움을 준다. 현재 운영 중인 인덱스는 Logstash 자체에서 인덱스 설정을 통해 Rollover 하고 있으므로 위와 같이 사용하지만, 언젠가 ILM 의 Auto Rollover 기능을 정말로 사용하게 될 때에는 어떻게 해야 하는가? 그 방법을 간단하게 아래에 작성하기로 한다.

### Logstash Pipeline

```json
input { ... }
filter { ... }
output {
  elasticsearch {
    hosts => ["<Node#01>:9200",...,"<Node#NN>:9200"]
    ilm_enabled => auto
    ilm_pattern => "{now/d}-000001"
    ilm_policy => "test-policy"
    ilm_rollover_alias => "test"
  }
}
```

- **ilm_enabled** : ILM 기능을 사용할 것인가 설정하는 옵션으로 기본값은 auto. auto 값인 경우에는, **데이터를 전송할 Elasticsearch 의 버전이 7.0 이상인 동시에 ILM 서비스를 사용하도록 유효화되어있을 때에는 Logstash 에서도 ILM 을 사용 가능하게** 설정하고, 해당 조건 외의 경우에는 ILM 을 무효화시킨다.
- **ilm_pattern** : 인덱스가 자동으로 롤오버될 때 생성될 인덱스 이름의 패턴을 정한다. 위의 코드는 기본값으로, 저대로 실행할 경우 인덱스는 <인덱스>-2022.02.16-000001, <인덱스>-2022.02.16-000002 ... 순서대로 만들어질 것이다.
- **ilm_rollover_alias** : 인덱스 이름이 될 문자열을 여기 적어주면 된다. 즉, 인덱스 이름을 test-2022.02.16-000001 같이 만들고 싶으면 이 옵션값에 ‘test’ 라고 적어주면 된다. 이 옵션은 `index` 옵션보다 강하므로, `ilm_rollover_alias` 옵션과 `index` 옵션이 함께 있을 경우 **우선적으로 `ilm_rollover_alias` 옵션을 따르게 된다.**

### ILM Policy

```json
PUT _ilm/policy/test-policy
{
  "policy": {
    "phases": {
      "hot": {                                
        "actions": {
##### 아래 rollover 옵션이 들어가 있다면 auto rollover 기능을 사용하겠다는 의미
          "rollover": {
            "max_primary_shard_size": "50GB", 
            "max_age": "30d"
          }
        }
      },
      "delete": {
        "min_age": "90d",                     
        "actions": {
          "delete": {}                        
        }
      }
    }
  }
}
```

### Index Templates

```json
PUT _index_template/test_template
{
  "index_patterns": ["test-*"],                 
  "template": {
    "settings": {
      "number_of_shards": 3,
      "number_of_replicas": 1,
      "index.lifecycle.name": "test-policy",     
      "index.lifecycle.rollover_alias": "test"    
    }
  }
}
```

`index.lifecycle.rollover_alias` 옵션은 Auto Rollover 를 사용하기 위해서는 필수적으로 지정해 주어야 하는 옵션이다. Logstash Pipeline 에서 지정한 `ilm_rollover_alias` 값과 동일하게 설정해 주어야 한다.

여기까지 설정해 주고 Logstash 에서 ES 로 데이터를 적재하기 시작하면, test-2022.02.16-000001 이라는 인덱스명으로 인덱싱이 시작될 것이다. 여기서

### 주의! 인덱스의 쓰기 권한 부여는 반드시 처음 생성된 ‘인덱스’ 에 설정한다

인덱스가 만들어지고 데이터가 적재됨에 따라, 설정한 rollover policy 를 충족하게 되면 자동으로 인덱스가 Rollover 된다. 즉, 새로운 인덱스 test-2022.02.16-000002 가 생성되어 데이터 적재를 이어받게 되고 test-2022.02.16-000001 인덱스에는 더 이상 데이터가 적재되지 않는 것.

이런 사이클이 돌아갈 때 중요한 것은, 새로운 인덱스가 만들어지면 그 인덱스에만 데이터가 적재(Write)되어야 하고 이전 인덱스에는 신규 데이터가 들어가면(Write) 안된다는 것이다. 즉, **Rollover 되는 인덱스는 항상 최신 인덱스에만 쓰기(Write) 권한이 부여되어야 한다는 것**이다.

Elasticsearch 에서는 Auto Rollover Policy 가 적용된 첫 인덱스에 write 전용 권한을 부여하면 다음 인덱스 rollover 가 일어날 때 자동으로 그 권한을 위임할 수 있는 매커니즘이 들어가 있다. 우리는 이 설정만 해 주면 된다.

```json
PUT test-2022.02.16-000001
{
  "aliases": {
    "timeseries": {
      "is_write_index": true
    }
  }
}
```

혹시라도 착각해서 이 설정을 인덱스 템플릿에 준다거나, Auto Rollover 가 아닌 Logstash pipeline 의 `index` 옵션으로 rollover 되는 인덱스에 설정하게 되면(요컨대 위의 설정이 복수의 인덱스에 들어가게 되면), 쓰기 옵션이 여러 인덱스에 설정되는 문제가 생긴다. 이러면 가장 처음에 생긴 인덱스에만 데이터가 Write 되고 이후 인덱스는 사용 불가한 현상이 벌어지므로 주의가 필요하다. 

> Elasticsearch alias has more than one write index (not a duplicate of any other question)
> 

이런 에러가 출력되고 데이터 적재가 안되는 이슈를 발생시킨다.

~~왜 강조했느냐... 이 설정을 인덱스 템플릿에 줘서 인덱스 패턴에 해당되는 모든 인덱스에 쓰기 권한을 부여하려 드는 탓에 이후 인덱스는 생성조차 되지 않았던 삽질을 무려 회사 무리의 長이 저질렀고 애꿎은 내가 욕먹었었기 때문~~

## ILM 바꾸기

위 삽질로 ILM 설정을 바꿨는데, 이미 기존 ILM 설정이 들어간 인덱스들에 대해서도 새로운 ILM 으로 바꾸고 싶을 때에는 어떻게 하는가?

### 기존 ILM 설정 삭제하기

```json
POST <인덱스명, wildcard 사용 가능>/_ilm/remove
```

### 새로운 ILM 설정하기

```json
PUT <인덱스명, wildcard 사용 가능>/_settings
{
  "index" : {
    "lifecycle" : {
      "name" : "test-policy"
    }
  }
}
```

## 참고

- [https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started-index-lifecycle-management.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started-index-lifecycle-management.html)
- [https://www.elastic.co/guide/en/logstash/current/plugins-outputs-elasticsearch.html#plugins-outputs-elasticsearch-ilm](https://www.elastic.co/guide/en/logstash/current/plugins-outputs-elasticsearch.html#plugins-outputs-elasticsearch-ilm)
