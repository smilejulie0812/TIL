## 드디어 업무 대화에 제대로 등장했다, Load Balancing
신규 ES 시스템을 구축해서 기존 시스템에 걸린 도메인을 이관하는 과정에서 다소 충격적인 사실을 알게 되었다. 

당연히 Elasticsearch 노드 전체가 LB 로 묶여서 도메인이 걸려있을 줄 알았는데 사실은 Coordinating 전용 서버 1대만 도메인을 걸어서 모든 트래픽이 그 노드로 집중되고 있었던 것.

이번 시스템은 전체 노드에 role 분배 없이 동일하게 사용하기 때문에, Load Balancing 으로 노드를 묶어서 사용하고자 설정 변경을 요청하게 되었고, 그 연장선으로 **Load Balancing 즉 부하분산**에 대한 기본 지식을 점검하게 되었다.

### Load Balancing

쏟아지는 트래픽을 복수의 **서버**로 분산해주는 기술

서버에 가해지는 부하(Load)를 분산(Balancing)해주는 장치 혹은 기술을 통칭하는 용어

각각의 서버에 가해지는 트래픽을 적절하게 분배해줌으로서 각 서버가 최적의 퍼포먼스를 보일 수 있도록 해 준다

- 서버 풀(Pool) : 이 분산 네트워크를 구성하는 서버의 그룹 혹은 리스트. 그러니까 로드밸런싱을 함께 할 가족 서버 군단 같은 느낌.

### Load Balancing Type

- 라운드 로빈 : 클라이언트로부터 요청이 들어온 순서대로 서버에 배분 → 서버가 같은 리소스를 가지고 있고, 각 요청의 세션(서버와 연결하는 것)이 짧은 경우 적합하다
- 가중 라운드 로빈 : 각 서버마다 가중치를 설정한다 → 가중치가 높은 서버에 우선적으로 요청을 배분 → 각 서버마다 트래픽 처리 능력이 다를 때 적합하다
- IP 해시 방식 : 서버 - 클라이언트 IP 를 매핑하여 담당 서버에 배분하도록
- 최소 연결 방식 : 가장 적은 연결상태인 서버를 찾아내어 배분 → 트래픽이 일정하지 않을 때 적합하다
- 최소 응답 시간 방식 : 서버의 연결 상태와 응답 시간을 고려해서 배분

### 참고

[https://m.post.naver.com/viewer/postView.nhn?volumeNo=27046347&memberNo=2521903](https://m.post.naver.com/viewer/postView.nhn?volumeNo=27046347&memberNo=2521903)

설명이 굉장히 쉽게 되어 있는 블로그여서 앞으로도 자주 들어가서 읽어봐야겠다.

### 막간 도메인 상식

당연하긴 한데, 도메인 신청할 땐 IP 주소 - 도메인 짝으로 신청하는 거지 IP 주소 - 도메인:포트 신청은 불가능하다(~~네트워크 지식 어떻게 된겨~~)

### 과제

Load Balancing 은 OSI Layer L4, L7 에서 움직인다는(정확히 말하면 L4 스위치랑 L7 스위치가 해주는 거) 점은 알고 있는데, 각각의 Load Balacing 동작이 어떻게 달라지는지까지는 정확히 파악 못하고 있지 않나. 이 부분 추가 공부가 필요한 듯(NLB, ALB 의 차이점).

그리고 로드밸런싱의 동작 방식을 좀 더 깊이 알아두면 추후 도움이 많이 될 것 같기도 하다.

로드밸런서를 HA 구성으로 하여 장애에 대비하는 구성도 알아두면 좋을 듯.

[https://dev.classmethod.jp/articles/load-balancing-types-and-algorithm/](https://dev.classmethod.jp/articles/load-balancing-types-and-algorithm/) 랑  
[https://prohannah.tistory.com/62](https://prohannah.tistory.com/62) 여기 참고!
