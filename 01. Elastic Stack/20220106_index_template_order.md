## Index Template 에 Order 를 지정하자
index template 온갖 걸 만들어서 지정하고 매핑 정보 일일이 엑셀에 리스트화하느라 눈이 빠질 것 같았던 그런 시절도 있었는데...  
이번에 ES 클러스터 옮기면서 새로 index template 을 만들며 새삼스럽게 order 숫자 적용 순서가 헷갈려서 가볍게 찾아보고 나서 정리해 두는 것.  
결론부터 말하자면, **Order 숫자가 작으면 작을수록 먼저 적용된다**는 것이다.

```json
PUT /_template/template_1
{
    "index_patterns" : ["*"],
    "order" : 0,
    "settings" : {
        "number_of_shards" : 1
    },
    "mappings" : {
        "_doc" : {
            "_source" : { "enabled" : false }
        }
    }
}

PUT /_template/template_2
{
    "index_patterns" : ["te*"],
    "order" : 1,
    "settings" : {
        "number_of_shards" : 1
    },
    "mappings" : {
        "_doc" : {
            "_source" : { "enabled" : true }
        }
    }
}
```

위의 예시(귀찮아서 elastic 사에서 들어준 규칙 그대로 가져옴)를 보면,  
template_1 은 order 가 0 인 인덱스 패턴 * 이고, template_2 는 order 가 1 이고 인덱스 패턴이 te* 인데,  
이 경우 tem 이라는 인덱스가 템플릿을 타는 순서는 먼저 template_1 을 거친 후 template_2 를 거친다.

그래서 _source 에 저장되는 설정이 template_1 을 탈 때에는 true 가 되지만 template_2 를 타면서 다시 false 로 바뀌게 되어 결정적으로는 설정이 false 로 들어가게 되는 것이다.

그렇기 때문에, 혹시라도 하나의 인덱스에 대해 인덱스 템플릿을 복수로 설정하게 될 경우가 있다면(예를 들어 전체 인덱스에 대해 공통적인 템플릿을 만들고 이후 각각의 인덱스 패턴에 따라 추가 템플릿을 작성할 경우),  
공통 인덱스에 대한 템플릿의 order 는 0 으로, 개별 인덱스에 대한 템플릿의 order 는 1로 설정하면 된다.

### 참고
https://www.elastic.co/guide/en/elasticsearch/reference/6.8/indices-templates.html
