# IP 와 Domain 설정 이모저모
하나의 VIP 에 여러 도메인을 설정할 수 있는가? 가능하다! 에서 출발하여 찾아본 내용 정리하기

## 1:N 의 IP:Domain 설정하기

하나의 서버 혹은 IP 주소에 대해 복수의 Domain 주소를 설정은 로드밸런서에서 하나 싶었는데 **DNS 에서 서브도메인 추가 설정 → Apache 웹 서버에서 가상호스트 추가 설정**으로 가능하다고 한다.

### Name 서버에서 서브도메인 추가하기

- /etc/named.conf 에서 메인 도메인의 zone 파일을 확인

```bash
zone "julie1.com" { type master; file "named.zone"; };
zone "julie2.com" { type master; file "named.zone"; }; ### <- 확인
```

- 1에서 확인한 zone 파일 디렉토리로 이동하여, 해당 zone 파일을 편집모드로 열기

```bash
cd /var/named
vi named.zone
```

- 하단에 서브도메인으로 설정할 내용을 추가

```bash
ftp                                IN      A       192.168.34.21
www                                IN      A       192.168.34.21
telnet                             IN      A       192.168.34.21
mail                               IN      A       192.168.34.21
pop3                               IN      A       192.168.34.21
### 아래에 추가할 서브도메인 작성
add                                IN      A       192.168.34.21
```

- 네임서버 재시작 및 확인

```bash
systemctl restart named
nslookup add.julie2.com
```

### Apache 서버에서 가상호스트 추가하기

- Apache 서버의 httpd.conf 파일에서 <VirtualHost> 추가

```bash
vi /etc/httpd/conf/httpd.conf
### 아래 VirtualHost 정보 추가
< VirtualHost 192.168.34.21 >
      ServerAdmin webmaster@julie2.com
      ServerName add.julie2.com
      ServerAlias www.add.domain.com
      DocumentRoot /home/shop/public_html
      ScriptAlias /cgi-bin /home/shop/public_html/cgi-bin
    < /VirtualHost >
```

- 아파치 서버 재시작

```bash
systemctl restart httpd
```

## N:1 의 IP:Hostname 설정하기

원래 궁금했던 내용은 위의 경우였는데, 반대로 하나의 도메인 주소에 여러 IP 주소를 붙일 수 있는지 궁금해졌다.

검색한 결과 **DNS 로드밸런싱** 기술을 통해 가능하다고 해서 간단하게 정리해 두도록 한다(물론 보통의 기업에서는 별도의 로드밸런싱 장비를 사용하겠지만).

> **DNS 로드밸런싱**이란? DNS 서버에서 도메인 정보를 조회할 때 매핑된 여러 IP 주소로 안내하여 트래픽을 분산시키는 기법. 별도 소프트웨어나 하드웨어로 된 로드밸런서를 사용하지 않아도 가능하다.
> 

> **DNS 라운드로빈**이란? DNS 로드밸런싱이 움직이는 방식으로, 유저가 특정 도메인으로 액세스하고자 할 때 해당 도메인으로 연결된 여러 IP 주소들 중에서 라운드 로빈 방식으로 IP 를 리턴하여 액세스하게 되는 방식
> 

### Name 서버 zone 파일에 IP 추가하기

- 해당 도메인의 zone 파일 수정

```bash
vi /var/named/julie2.com.zone
$TTL 3H
@         IN       SOA     ns.julie2.com.   admin.julie2.com. (
                                                 1  ;       serial
                                                 1D      ;       refresh
                                                 1H      ;       retry
                                                 1W      ;       expire
                                                 3H )    ;       minimum

          IN       NS      ns.julie2.com.
          1   IN   A       192.168.34.21
          1   IN   A       192.168.34.12
ns        IN       A       192.168.34.12
www       1  IN    A       192.168.34.21
www       1  IN    A       192.168.34.12
```

- 설정한 zone 파일 수정 및 네임서버 재시작

```bash
### 위에서 설정한 zone 파일의 syntax 에러 점검
named-checkzone julie2.com /var/named/julie2.com.zone
# OK 출력되면 문제 없음
### 네임서버 재시작
systemctl restart named 
```

### Apache 서버 index 파일에 추가 IP 정보 작성하기

- index.html 파일에 서버 정보 추가

```bash
cd /var/www/html
vi index.html
### 서버별로 내용 입력 : 1번 서버, 2번 서버 ...
<html><body><h1>{IPaddress1}</h1>
</body></html>
```

- 아파치 서버 재시작 및 설정 확인

```bash
systemctl restart httpd
nslookup julie.com
# Addresses: 192.168.34.21
#            192.168.34.12
# 임을 확인하기
```

## 참고

- [https://www.sharedit.co.kr/qnaboards/23445](https://www.sharedit.co.kr/qnaboards/23445)
- [https://cheolgoon.tistory.com/entry/리눅스-서브도메인-셋팅](https://cheolgoon.tistory.com/entry/%EB%A6%AC%EB%88%85%EC%8A%A4-%EC%84%9C%EB%B8%8C%EB%8F%84%EB%A9%94%EC%9D%B8-%EC%85%8B%ED%8C%85)
- [https://blog.jiniworld.me/27#a03](https://blog.jiniworld.me/27#a03)
- [https://hoing.io/archives/8607](https://hoing.io/archives/8607)
