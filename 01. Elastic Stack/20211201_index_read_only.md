## Read-only Index 구하기
### 문제
Kibana 웹페이지 상에서 Search/Visualization/Dashboard 로 신규 저장 및 기존 정보를 수정하고자 할 때 Forbidden 에러로 내용이 저장되지 않는 현상
```
### 디스크 용량 부족으로 인덱스에 데이터 적재가 불가한 상황
[2021-11-03T00:02:19,972][INFO ][o.e.c.r.a.DiskThresholdMonitor] [<HOSTNAME>] low disk watermark [85%] exceeded on [cRg5XcJjS2KMYtmEpKyrdg][<HOSTNAME>][/<DATAPATH>/data/vnodes/0] free: 11.4gb[14.5%], replicas will not be assigned to this node
 
### 인덱스가 read-only 로 바뀌어 데이터 적재 불가
[2021-11-03T14:27:14,457][WARN ][o.e.x.m.e.l.LocalExporter] [<HOSTNAME>] unexpected error while indexing monitoring document
org.elasticsearch.xpack.monitoring.exporter.ExportException: ClusterBlockException[blocked by: [FORBIDDEN/12/index read-only / allow delete (api)];]
```
### 원인
Kibana 가 움직이는 서버 상에서 용량에 비해 많은 데이터가 들어오게 되면 자동으로 read-only 모드로 전환하는 기능이 실행된다고 한다(ES 역시 같은 기능 실행).  
실제로 GET _settings 로 인덱스 현황을 살펴보니, 대부분의 인덱스에 해당 설정이 유효화되어 있었다.  
현재 Kibana 가 올라가 있는 서버의 디스크를 확인해 보니 60% 이상 사용하고 있었고, 이것이 인덱스가 read-only 로 전환된 이유가 되지 않았나 예상해본다.  
(디스크를 얼마나 사용했을 때 모드가 read-only 모드로 전환되는지는 차후 조사하여 내용 추가할 예정 -> 이거 이미 올렸으니 연동하자!)
### 움직임
**index.blocks.read_only_allow_delete 값의 기본값은 false**
정상적으로 움직이고 있다가 데이터 노드 쪽 서버의 디스크가 85% 이상의 사용률로 바뀔 경우, ES 에서 자체적으로 해당 인덱스 setting 을 index.blocks.read_only_allow_delete : true 로 변환하여 인덱스를 read-only 로 변환한다.  
그리고 서버 디스크가 다시 85% 이하로 떨어지면 ES 가 자동으로 index.blocks.read_only_allow_delete : false 로 변환하여 read_only 인덱스 설정을 해제한다.
### 해결
아래 API 로 해결
```JSON
PUT .kibana/_settings
{
  "index": {
    "blocks": {
      "read_only_allow_delete": "false"
    }
  }
}
```
만약 위의 API 로 해결되지 않을 때에는 PUT /_settings 로 바꾸어 전체 대상으로 read-only 모드를 비활성화한다.
### 해결 후
read_only_allow_delete 를 false 로 일시적인 처리를 마친 후에는 아래의 설정으로 기존 설정을 리셋한다.
```JSON
PUT .kibana/_settings
{
  "index": {
    "blocks": {
      "read_only_allow_delete": null
    }
  }
}
```
### 참고
* https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-cluster.html#cluster-routing-flood-stage
* https://dev-yeon.tistory.com/12
* https://qummi.tistory.com/26
