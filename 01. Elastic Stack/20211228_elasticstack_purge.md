# Elastic Stack Purge
테스트 환경에 일시적으로 설치한 패키지를 삭제하거나, 버전 업데이트를 위한 전처리 과정으로 삭제가 필요할 경우를 위해 간단하게 정리해 둔다.

```bash
sudo apt-get purge elasticsearch
sudo apt-get purge kibana
```

purge 명령어로 삭제하면 패키지 설치할 때 생성된 파일이나 디렉토리는 함께 삭제된다. 예를 들면 아래와 같은 디렉토리.

- /etc 아래 설정 파일용 디렉토리 ※ 다만 /etc/kibana 는 keystore 파일이 남아있어 삭제되지 않음
- /var/log 아래 로그 파일 보관용 디렉토리
- /usr/share 아래 실행 파일 보관용 디렉토리
- /etc/default 아래 디폴트 설정(변수) 파일

다만 /var/lib 아래의 데이터 저장 디렉토리의 경우, 설치 이후 새롭게 만들어진 데이터가 생성되는 장소이기 때문에 purge 명령어로도 사라지지 않으므로 수동으로 삭제해준다.

```bash
### rm -rf 옵션 명령어는 확인 절차 없이 바로 삭제하는 명령어이므로 항상 주의한다.
rm -rf /etc/kibana
rm -rf /var/lib/elasticsearch
rm -rf /var/lib/kibana
```

[https://blog.naver.com/PostView.nhn?blogId=indy9052&logNo=221387817807&categoryNo=66&parentCategoryNo=0&viewDate=&currentPage=1&postListTopCurrentPage=1&from=postView&userTopListOpen=true&userTopListCount=5&userTopListManageOpen=false&userTopListCurrentPage=1](https://blog.naver.com/PostView.nhn?blogId=indy9052&logNo=221387817807&categoryNo=66&parentCategoryNo=0&viewDate=&currentPage=1&postListTopCurrentPage=1&from=postView&userTopListOpen=true&userTopListCount=5&userTopListManageOpen=false&userTopListCurrentPage=1)
