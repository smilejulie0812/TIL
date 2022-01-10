[https://wikidocs.net/130113](https://wikidocs.net/130113)

[https://velog.io/@hanblueblue/번역-Ansible](https://velog.io/@hanblueblue/%EB%B2%88%EC%97%AD-Ansible)

### 정의와 대표적인 특징

- IT 자동화 도구로, 시스템 구성과 소프트웨어 배포 등의 역할을 수행한다
- SSH 연결
- hosts

### playbook 구성

- 플레이북이란
    - 한 번에 하나 이상의 작업을 하는 기능
    - 한 번에 수많은 장비에 수많은 액션을 수행할 수 있다
    - yaml 파일로 표현 → 표준 야믈 파서 사용하여 아믈의 모든 기능 사용 가능
- 타깃 부분
    - 플레이가 실행될 장비와 어떻게 플레이가 실행될 것인지 정의
- 변수 부분
    - 플레이에서 실행할 때 쓰일 사용 가능한 변수를 정의
    - vars, vars_files, vars_prompt
- 태스크 부분
    - 실행하고 싶은 모든 모듈을 순서대로 열거
- 핸들러
    - 기본적으로 테스크에 의해 호출되지 않으면 실행되지 않음
    - 태스크가 뭔가를 변경했던 레코드에서 태스크가 호출될 때만 핸들러가 호출된다

### 모듈

- 모듈이란 그리고 기본 모듈
- template 모듈, wait_for 모듈, group_by 모듈

### 조건절, 태그

### 롤

- 특정 작업을 수행하기 위한 플레이북의 그룹화된 저장소
- 롤 폴더에 변수, 파일, 태스크, 템플릿, 핸들러 위치할 수 있다

| 디렉토리 | 설명 |
| --- | --- |
| tasks | * 태스크 롤의 태스크 목록을 포함하는 main.ymal 파일을 포함
* 태스크 폴더의 파일을 나누어 작성 가능
* 많은 태스크를 독립된 파일로 나누어 태스크 인클루드의 기능을 사용할 수 있음 |
| files | 사용되는 롤의 파일을 위치시킴 |
| templates | template 모듈이 자동으로 templates 롤의 진자2 템플릿을 찾는 디렉토리 |
| handlers | 롤의 핸들러를 상세한 mail.yml 파일ㅇ르 포함 |
| vars | 롤의 변수를 포함하는 main.yml 파일을 포함 |
| meta | 롤을 위한 설정과 설정의 의존 목록을 포함할 수 있다 |
| default | 롤에 적용할 선택적인 변수를 설정 |

### 실행 명령어

### 다른 배포 툴과의 비교 : chef, puppet, terraform 까지

### 예시 : Filebeat Install 및 config 배부

### 참고 URL

[https://blog.tunarider.net/hands-on/terraform-hands-on/](https://blog.tunarider.net/hands-on/terraform-hands-on/)

[https://www.elastic.co/kr/blog/automate-all-the-things-terraform-ansible-elastic-cloud-enterprise-ece](https://www.elastic.co/kr/blog/automate-all-the-things-terraform-ansible-elastic-cloud-enterprise-ece)

[https://swalloow.github.io/tf-tips/](https://swalloow.github.io/tf-tips/)
