## MySQL Login Path 설정하기

Elastalert 서버에서 MySQL 서버로 프로세스 count 정보를 전송해서 모니터링 설정을 걸어놔야 할 일이 생겼다.  
데이터 수집하는 스크립트야 bash 로 간단하게 만들었는데, 문제는 환경 설정.

비 DB 서버에서 MySQL 서버로 데이터를 전송하기 위해서 필요한 것이 바로 MySQL Login Path 옵션이다.  
MySQL 5.6 버전부터 보안 이슈로 인해 패스워드를 포함한 명령어 입력이 불가능해졌고(그 이전에 명령어에 패스워드 넣어서 보내는 자체가 문제 아닌가)  
이를 해결하기 위해 Login Path 옵션이 불가피해졌다고 한다.  

참고) 요런 로그가 출력된다고
```
Warning: using a password on the command line interface can be insecure. 
```

해야 할 일은 다음과 같다.
1. 방화벽 오픈 -> MySQL 서버 IP:3306
2. 서버에 mysql-client 설치
    1. mysql -V
    2. sudo apt-get install mysql-client-core-5.6 ### 굳이 안해도 mysql-client 설치할 때 사전 인스톨 됨
    3. sudo apt-get install mysql-client
3. mysql_config_editor 파일을 이용하여 login path 설정
    1. cd /usr/bin
    2. mysql_config_editor --help
    3. /usr/bin/mysql_config_editor set --login-path=<login path name> --host=<mysql server ip address> --user=<username> --port=3306 --password
    4. mysql_config_editor print --login-path=<login path name>
4. 클라이언트에서 서버로 데이터 전송
```bash
#!/bin/bash
HTNM="<호스트명>"
PRO_CNT=<카운트수>
MON_TS=$(date +%s)

TEST_PRO_CNT=$(ps -ef | grep "<카운트용 문자열>" | wc -l)

/usr/bin/mysql --login-path=n<login path 이름> -Be "use <데이터베이스명>; call <DB 테이블 적재용 프로시저>('$HTNM', '$PRO_CNT', '$TEST_PRO_CNT', '$MON_TS');"
```
  
### 참고
http://www.irgroup.org/mysql-login-path/
