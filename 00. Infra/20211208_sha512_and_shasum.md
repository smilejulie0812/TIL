## SHA512 파일의 역할

### 배경
Elasticsearch 를 설치할 때 공식 페이지에서 다음과 같은 명령어를 가이드해준다.
```bash
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-<버전>-amd64.deb
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-<버전>-amd64.deb.sha512
shasum -a 512 -c elasticsearch-<버전>-amd64.deb.sha512
sudo dpkg -i elasticsearch-<버전>-amd64.deb
```
그냥 생각없이 쓰던 명령어이긴 한데, 솔직히 패키지 파일이랑 별도로 sha512 파일은 왜 따로 다운받아야 하는지,  
저 shasum 명령어가 무엇을 의미하는지, 등등이 궁금해져서 내 의식의 흐름과 구글 선생님의 지도에 따라 찾아보게 되었다.  
그 내용을 정리해 두는 공간.

### 해시함수
해시함수는 어떤 임의의 길이로 주어진 데이터를 고정된 길이의 데이터로 매핑하는 함수이다.  
그리고 그 해시 함수에 의해 얻을 수 있는 고정된 길이의 데이터를 해시값, 해시코드, 해시체크섬(다 줄여서 해시)라고 불린다.  

해시함수는 데이터베이스 검색 측면에서, 큰 파일에서 중복되는 레코드를 찾을 수 있어 빠른 검색이 필요할 때 도움이 된다고 한다.  
그런데 내가 집중하고 싶은 부분은, 암호학 측면에서 이 해시함수가 사용될 때, 보안 설정에 용이하다는 점이다.  
해시 함수를 통해 얻은 결과값을 가지고 원래 데이터를 알아내는 것이 매우 힘들다는 특징을 가지고 있으며,  
메시지를 전송할 때 그 메시지에 대한 무결설을 보장해준다(해시 함수의 결과값은 항상 유일무이하다).  

즉, 해시 함수는 보안 측면에서 강력한 도구라고 할 수 있다.

### SHA 알고리즘
해시 함수에서 가장 보편화된 알고리즘이 바로 SHA 알고리즘이다(MD5 알고리즘도 많이 쓰이지만 SHA 가 더 강력하다고...)  
그 중에서도 SHA-512 는 입력 메시지의 길이가 2^128 까지이고, 이 메시지로부터 512 bit 의 해시값을 출력해내는 함수이다.  

~~(알고리즘까지 다루게 되면 삼천포로 빠질 것 같으니 패스하고)~~ SHA-512 가 매우 강력한 보안 기능을 지닌 함수라는 사실은 깨달을 수 있다.  

### shasum 명령어로 해시 값 체크
배경 지식 조사가 길었는데 본론으로 들어가 보자면, sha512 파일은 위의 SHA-512 알고리즘을 통해 얻은 해시 암호 코드 파일이다.  
즉 Elasticsearch 는 데비안 패키지 파일과 sha512 해시 암호 코드 파일이 별도로 존재한다.  
(참고로 Kibana 는 데비안 패키지 파일 내에 sha512 파일이 함께 존재함). 

shasum 명령어는 해당 퍄일의 SHA 해시값을 출력하거나 체크할 수 있는 명령어이다.  
즉, 이번 경우에 빗대어 설명하자면 Elasticsearch 데비안 패키지 파일과 해시 암호 파일을 비교하기 위해 사용하는 명령어이다.  

만약 wget 명령어로 패키지 파일을 다운로드하는 과정에서 네트워크 이슈 등으로 정상적인 다운로드가 이루어지지 않았을 경우 어떻게 될까?  
당연히 다운로드 받던 파일이 손상될 것이다. 물론 손상된 Elasticsearch 파일을 설치하면 문제가 생기는 것은 필연적일 테고.  

그걸 막기 위해 진행하는 것이 바로 shasum 이다.  
해시값을 비교하여 정상적인 해시값인지 확인해서, 파일이 손상되거나 변형된 것이 아닌지 확인하는 과정인 것.  

위의 shasum 명령어에서 쓰인 두 옵션의 기능을 간단히 정리하자면,  
* **-a** : --algorithm 의 약자로 SHA 알고리즘의 종류를 적는다.
다만 이 옵션은 sha512sum, sha256sum 과 같은 명령어로 대체할 수 있을 것 같다.
* **-c** : --check 의 약자로 체크섬 파일로부터 해시값을 읽어와 체크하는 기능이다.
이번 경우에는 Elasticsearch 패키지 파일의 전용 sha512 파일로부터 값을 읽어오도록 설정되었다.

원래 Elasticsearch 6.2 버전까지는 shasum 과정이 생략되어 있었는데,  
6.3 버전부터 자체에 x-pack 이 기본설치에 포함되면서 이 해시값 체크 과정이 포함되었다고 한다.  
(일일이 x-pack tar 파일 풀어줘야 했던 6.2 이전 버전 설치가 아련히 떠오르는...)

### 참고
* https://m.blog.naver.com/vjhh0712v/221446598465
* https://m.blog.naver.com/vjhh0712v/221453210356
* https://blog.naver.com/PostView.nhn?blogId=username1103&logNo=222113930052
* https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=stk001&logNo=120029381802
* https://janito.tistory.com/8
* https://thersfy.tistory.com/m/5
* https://whatext.com/ko/sha512
