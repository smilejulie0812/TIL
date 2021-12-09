## Elasticsearch 버전에 따른 JAVA 설정

Elasticsearch 는 JAVA 기반의 솔루션이기 때문에 사용하는 데 당연히 JVM 이 필요하다.  
최소 Java 8 이상이 필요하며, OpenJDK 와 Oracle JDK 만을 지원하기 때문에 보통 무료 버전인 OpenJDK 를 사용한다.  

근데 Elasticsearch 의 버전에 따라 사용할 JVM 을 설정하는 방법이 달라진다는 사실을 이번에 알게 되어 내용을 정리한다.  

### 버전 6 이하
버전 6 이하의 ES 에서 사용할 JDK 는 일반적인 방법으로 설치하면 된다.
```bash
sudo apt-get install openjdk-8-jdk	# OpenJDK 1.8.0 버전을 설치할 경우
java -version

```

여기서 한 가지 주의할 점은, ES 를 설치하기 전에 사용할 JDK 를 설치해 주어야 Java 버전의 충돌 없이 설치할 수 있다는 것이다.

만약 ES 를 설치한 후 JDK 를 설치해서 그 JDK 를 사용하고자 하면,  
이미 ES 가 설치될 때 기존의 JDK 경로를 바라보는 설정이 들어가기 때문에 버전 충돌 이슈가 발생할 수 있다.

물론, 이 경우에는 환경변수의 JAVA_HOME 을 명시해 주는 것으로 해결된다.
```bash
### 아래 파일에 기재해두면 서버를 재시작해도 설정이 적용된다
vi /etc/profile
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64

### 해당 터미널에서 로그아웃했다가 다시 로그인
echo $JAVA_HOME
### JAVA_HOME 에서 설정한 JDK 버전 확인
$JAVA_HOME/bin/javac -version
```

### 버전 7
버전 7 이상의 ES 는 설정이 조금 복잡해진다.  
이 때부터 Elasticsearch 패키지 안에 번들 JDK 가 포함되어 있고, 기동할 때 그 번들 JDK 를 사용하도록 기본 설정이 들어가 있기 때문이다.  
번들 JDK 는 ES 를 설치할 때 아래 경로에 자동으로 생성된다.
```bash
{Elasticsearch 설치 디렉토리}/elasticsearch/jdk
```

단순히 생각해보면 번들 JDK 를 사용하는 게 따로 JDK 설치가 필요 없어서 편할 것 같지만, 현실은 그렇게 간단하지 않다.  
제공되는 JDK 가 LTS 가 아닌 최신 버전 JDK 기 때문에, 안정적인 JDK 를 사용하고자 할 때에는 버전 6 의 경우와 같이. 
사용하고자 하는 JDK 를 설치한 후, ES 에서 해당 JDK 를 사용할 수 있도록 변수를 설정해 줄 필요가 있다.  

위의 내용과 같이 JDK 를 설치해 준 후, 아래 파일을 수정하는 방식으로 ES 가 바라보는 Java 디렉토리 경로를 꺾어준다.
```bash
vi /etc/default/elasticsearch
ES_JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
```

버전 7 부터 OS 전체의 JAVA_HOME 과 별도로 ES 전용으로 사용할 Java 디렉토리가 설정 가능한 변수 ES_JAVA_HOME 이 새로 생성되었다.  
즉, 버전 7 부터는 JAVA_HOME 정보를 바꿔준다고 해도 ES 가 해당 경로를 타지 않는다는 의미이다.  
위와 같이 전용 환경변수를 변경해 준 후 ES 를 시작해준다.

시스템이든 프로세스든 확인해주면 해당 ES 가 원하는 Java 버전으로 실행되고 있는지 확인할 수 있다.
```bash
### systemd 로 관리할 때. 출력되는 상태 정보에 사용하는 Java 경로가 함께 표시된다.
service elasticsearch status
### 가장 확실한 방법. 프로세스로 체크하면 실제 ES 가 실행되는 명령어가 출력되므로 쉽게 확인 가능하다.
ps -ef | grep elasticsearch
```

### 참고
* https://www.elastic.co/guide/kr/elasticsearch/reference/5.3/gs-installation.html
* https://www.elastic.co/guide/en/elasticsearch/reference/6.8/setup.html
* https://www.elastic.co/guide/en/elasticsearch/reference/7.16/setup.html
* https://zetawiki.com/wiki/%EB%A6%AC%EB%88%85%EC%8A%A4_$JAVA_HOME_%ED%99%98%EA%B2%BD%EB%B3%80%EC%88%98_%EC%84%A4%EC%A0%95
