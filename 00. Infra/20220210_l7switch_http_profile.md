# L7 Switch HTTP Profile

원래 1대의 Kafka Broker 로만 운영하던 것을 3대로 scale-out 하고 Kafka Cluster 로 묶는 작업을 했다.

여기서 한 가지 문제가 되었던 것이 도메인 주소이다. 원래 Broker 가 1대뿐이었어서 그 서버의 IP 에 도메인을 붙였었는데, 3대로 확장하고 나니 이 도메인 주소를 어떻게 붙여야 하는지 설정 변경이 필요했던 것이다.

해당 도메인 주소는 이 Kafka 를 사용하는 수많은 API 서버에서 메시지를 produce 하기 위해 사용하고 있었으므로, 반드시 이 도메인 주소를 살려서 API 서버 측에서 설정 변경이 없도록 설정해야 했다.

그래서, 3대의 Kafka Broker 앞에 Load Balacer 를 붙이고, 여기에 1개의 VIP 를 생성하여 그 VIP 에 도메인 주소를 붙이는 설정을 채택했다. 그러면 기존의 API 서버들은 도메인 주소를 통해 VIP 로 들어올 것이고, Load Balancer 가 3대의 Kafka Broker 들로 부하분산을 해 줄테니까.

Domain Port, Service Port (두 개의 의미 찾아서 정리해두기)까지 설정해서 구성은 완료했는데, 문제가 생겼다. API 서버 측에서 도메인 주소를 통해 Kafka 로 액세스하고자 하면 Disconneced 된다는 것.

![001.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d23a7aa2-8a25-414e-82d3-234e8651a8c5/001.png)

Kafka 쪽 설정파일인 [server.properties](http://server.properties) 에 추가 설정이 필요한가 등등 많은 삽질을 했지만 원인은 네트워크 쪽에 있었다. Kafka 에 연결하는 설정이기 때문에 웹 설정 등에서 사용하는 HTTP Profile 설정이 필요하지 않은데(그보다 설정하면 안되는데) L7 스위치에서 디폴트 설정값으로 들어가 있던 바람에 프로토콜 설정 충돌로 접근이 되지 않았던 것. 우선 네트워크 쪽에서 해당 설정값을 무효화해서 이슈는 해결했다.

## L7 Switch HTTP Profile

Switch 에서 Profile 이란 Virtual Server( VIP:Port ) 의 설정파일이라 할 수 있다.

즉, HTTP Profile 이면 L7 Swtich 에서밖에 불가능한 HTTP 프로토콜 수준의 로드밸런싱을 수행하기 위한 Virtual Server 의 세부 설정이라는 의미이다.

웹서버 등에 대한 로드밸런싱을 진행할 때에는 이 HTTP Profile 을 유효화하여 사용할 수 있도록 하면 편리하겠지만, 그렇지 않은 경우에도 이 HTTP Profile 이 유효화하다면 프로토콜 충돌로 인해 원래 설정이 정상적으로 움직이지 않을 것으로 예상된다.

실제로 위의 이슈도 L7 스위치의 서비스 정책에 디폴트 설정으로 HTTP profile 이 설정되어 있었는데, 이를 제거하지 않고 그대로 로드밸런싱 설정을 추가한 것이 원인이었던 것으로 판명났다(HTTP profile 은 L7 스위치의 트래픽 처리방식이 다르다고(L4 를 사용해도 되는 걸 L7 기능을 열어놔서 문제가 되었다는 말 아닌가)).

### 참고

- [https://aws-hyoh.tistory.com/entry/L4L7-로드밸런싱-쉽게-이해하기](https://aws-hyoh.tistory.com/entry/L4L7-%EB%A1%9C%EB%93%9C%EB%B0%B8%EB%9F%B0%EC%8B%B1-%EC%89%BD%EA%B2%8C-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0)
