## 맨날 쓰고도 맨날 헷갈리는 옵션값을 내 언어로 써보자
매일 설정해두고 쓰는 옵션이지만 혹시 대충 사용하고 있지는 않았는가 되돌아보며 정리해본다.

### indices.memory.index_buffer_size
인덱싱 데이터를 메모리에서 디스크로 기록하는 상한선 퍼센테이지.  
즉, 처음 메모리로 들어온 인덱싱 데이터가 설정한 퍼센트 이상으로 차올라서 버퍼가 차게 되면 이를 디스크로 옮겨 기록하는데, 그 기준이 되는 상한선 퍼센트를 설정하는 옵션이다.
기본값은 10%.

### bootstrap.memory_lock
보통은 Elasticsearch 초기설정할 때 ES 에서 사용할 heap memory 를 고정시키는데(물론 auto heap memory 로 사용할 수 있기는 하다), 같은 OS 상에서 다른 Java 프로그램이 이 설정된 ES 에서 잡은 heap size 를 사용할 수 없도록 점유하는 설정.  
기본값은 true 이며, 항상 true 로 놓고 사용하는 것을 권장한다.

### http.cors.allow-origin
다른 브라우저에서 이 ES 에 접근 가능하도록(http/https) 설정하는 옵션이다.  
이 옵션을 사용하려면 우선 http.cors.enabled 옵션을 true 로 설정해 주어야 한다.  
보통 해당 옵션은 * 로 설정하는데, 이건 모든 브라우저의 모든 URL 에서 액세스 가능하게 설정한다는 의미. 좀 더 제한을 두려면 정규표현식으로 옵션값을 넣어줄 필요가 있다.

### discovery.seed_hosts
어떤 같은 클러스터로 묶이기 위해 최소한으로 발견해야 할 노드의 정보를 여기에 적는다.  
즉, 어떤 Elasticsearch 노드가 처음 시작될 때, 이 옵션값에 적은 노드 정보를 찾으면, 그 노드 혹은 노드들로 구성된 클러스터에 Join 한다는 의미.  
보통 여기에는 클러스터를 구성할 최소한의 노드, 즉 마스터 노드(+후보 포함)를 설정한다.

### cluster.initial_matser_nodes
어떤 클러스터를 처음 구성할 때, 그 클러스터를 구성하는 노드들은 이 옵션에 쓰인 노드 정보를 보고, 이 옵션에 있는 노드들 중 하나의 노드를 선출해서 마스터 노드로 삼는다.  
즉, 어렵게 말할 필요 없이 여기 쓰인 노드들이 마스터 (후보) 노드인 것.