# 버전 6 Kibana 에 로그를 남길 때
버전 7은 데비안 패키지로 설치할 때 자동으로 /var/log/kibana 안에 로그를 출력해주는데,  
버전 6은 config 디폴트값이 없어서 수동으로 설정해줄 필요가 있어서 메모.  
(디폴트값은 stdout 으로 단순하게 뿌리는 방식임)

### config 파일 수정
* kibana.yml
```bash
#logging.dest: stdout
logging.dest: /var/log/kibana/kibana.log
```

### kibana 전용 디렉토리 생성 및 권한 설정
```bash
# /var/log 아래 전용 디렉토리 생성
mkdir /var/log/kibana
# 디렉토리 권한 설정
## 데비안 패키지로 설치할 경우 전용 유저인 kibana 유저가 생성되어 있지만, 아카이브로 풀어쓸 경우 유저를 직접 생성해 주어야 한다.
chown kibana:kibana /var/log/kibana
```

### Kibana 재시작 및 로그 출력 확인
```bash
# kibana 재시작
systemctl restart kibana
# 로그 파일 생성 확인
ls -la /var/log/kibana
```
