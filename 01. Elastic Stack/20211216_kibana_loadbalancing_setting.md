# Kibana Load Balancing Setting
## 서론

기존 시스템에서 복수의 Kibana 를 사용할 때 그 서버들의 IP 를 하나의 도메인으로 묶고, 따로 로드밸런서를 두는 구성을 채택하였다.

이러면 엔드 유저가 해당 도메인을 통해 Kibana 에 접속할 때, 로드밸런서가 여러 Kibana 중 한 곳으로 보내주고, 그 유저는 보내진 Kibana 에 접속한 후 세션이 끝날 때까지 그 Kibana 에서만 작업하는 움직임이었다.

그런데 새로운 형태의 구성이 들어왔다.

복수의 Kibana 를 사용하고, 그 서버들을 묶는 VIP 와 도메인이 존재하는데, 정작 사이에 로드밸런서 설정이 없는 구성이었던 것이다.

이런 상황이라면, 엔드 유저가 해당 도메인을 통해 Kibana 로 접속하게 되면 복수의 Kibana 서버 중 한 대에 접속하게 되지만,

다음 페이지로 넘어가게 될 때 그 Kibana 서버에서만 움직일 것이라는 보장이 없다.

만약 Elastic Stack 에 보안 설정이 걸려있지 않다면 이 상황이 크게 문제되지는 않을 것이다.

하지만 보안 설정이 걸려 있다면, 처음 도메인을 통해 Kibana01 서버에 액세스하여 로그인했다 하더라도,

이후 같은 도메인의 다른 페이지로 액세스를 시도할 경우, Kibana02 서버로 넘어가면 그 서버에서 로그인하지 않은 상태이기 때문에 각각의 Kibana 에서 쿠키 충돌이 일어나게 되어 사용이 어려워진다.

이 상황을 방지하기 위해, Elastic Stack 에 따로 로드밸런서가 달려 있지 않은 경우에는 Kibana 의 자체적인 로드 밸런싱 설정을 줄 수 있다.

<위의 내용 그림으로 그려서 설명하면 좋을듯>

## 복수의 Kibana 인스턴스에 대한 로드밸런싱 설정

### 각 Kibana 인스턴스에 고유의 UUID 를 부여

```bash
# uuid 설치
sudo apt-get install uuid
# uuid 부여
uuid
# kibana.yml 에 부여받은 uuid 설정
server.uuid
```

만약 한 서버에 복수의 인스턴스를 올렸다면(이럴 일이 많을까...? 한 서버에 여러 kibana 인스턴스를 올리는...)

```yaml
# kibana.yml
logging.dest
path.data
pid.file
server.port
```

### Kibana 의 암호화 키를 생성하여 로드밸런싱으로 묶을 인스턴스에 설정

```bash
# 암호화 키 부여받기 : 여러 키바나 인스턴스 중 한 곳에서만 받아도 된다
<kibana 설치 경로>/bin/kibana-encryption-keys generate
# kibana.yml : 모든 키바나 인스턴스에 동일한 값 적용
xpack.security.encryptionKey // 세션 정보 암호 해독
xpack.reporting.encryptionKey // 리포트 암호 해독
xpack.encryptedSavedObjects.encryptionKey // 저장된 오브젝트 암호 해독
```

### Kibana 인스턴스 간 쿠키값 설정

쿠키값을 일치시켜주면, 원활한 HA 가 가능해지기 때문에 일부 Kibana 인스턴스에 장애가 발생할 경우에도 세션을 활성 상태로 유지할 수 있다.

```yaml
# kibana.yml
xpack.security.enabled: true
xpack.security.cookieName: <쿠키값> // 쿠키값은 로드밸런싱할 키바나 인스턴스에 동일하게 준다
```

### 퍼블릭 URL(도메인) 설정

(인프라 측에서 해당 서버에 대한 공통 도메인을 줬다면 이 설정이 필요 없는지 궁금하다)

```yaml
# kibana.yml
server.publicBaseUrl: "<퍼블릭 URL 주소>"
## 참고로 해당 옵션은 kibana 7.11 이후 버전부터 사용 가능한 옵션이다.
```

### 키바나 재시작

```bash
service kibana restart
```

## 움직임

- 도메인으로 접속: 처음에 로그인 창 뜨고나서 들어가고 나면, 어디를 들어가더라도 로그인 없이 자연스럽게 사용 가능했다(쿠키값으로 로그인한 세션이 계속해서 유지됨)
- 각각의 IP 로 접속: Kibana01 의 IP 로 접속했을 때에는 해당 IP 에서만 이동하므로 문제가 없었고, 같은 브라우저에서 Kibana02 의 IP 로 접속했을 때에는 (당연하게도) 다시 로그인이 필요했다.

## 참고

- [https://www.elastic.co/guide/en/kibana/current/security-settings-kb.html](https://www.elastic.co/guide/en/kibana/current/security-settings-kb.html)
- [https://www.elastic.co/guide/en/kibana/current/production.html#accessing-load-balanced-kibana](https://www.elastic.co/guide/en/kibana/current/production.html#accessing-load-balanced-kibana)
- [https://www.elastic.co/guide/en/kibana/master/kibana-encryption-keys.html](https://www.elastic.co/guide/en/kibana/master/kibana-encryption-keys.html)
- [https://hahahoho5915.tistory.com/32](https://hahahoho5915.tistory.com/32)

## 과제

- 로드밸런싱에 대한 개괄적 이해
- 쿠키/세션 개념을 Kibana/Elasticsearch 에 빗대어 이해
- UUID 설치해서 부여받은 값과 해당 서버의 UUID(blkid 명령어를 이용해서 얻은) 의 차이점은? (차이 없을 거 같은데... 확인해보자)
