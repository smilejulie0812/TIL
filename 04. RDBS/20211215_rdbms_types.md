# 주요 RDB(Oracle, MySQL, PostgreSQL) 의 기능 비교(번역)
[https://qiita.com/kamihork/items/deeaf449254e3845311d](https://qiita.com/kamihork/items/deeaf449254e3845311d) 이 페이지 번역하면서 공부

## Oracle 의 주요 특징

Oracle 사가 개발, 판매하는 상용 RDBMS

### 행 레벨 잠금

복수의 사용자가 동일한 표의 각기 다른 행을 동시에 접근할 수 있어 성능면에서 탁월하다.

### 읽기 일관성

더티 리드(갱신 전의 정보와 갱신 후의 정보 양쪽을 취득하게 되는 현상)가 발생하지 않도록, 갱신 전의 데이터만을 취득할 수 있도록 한다.

### 견고성

Oracle Data Gaurd 라 불리우는, 재해나 데이터 파손 등의 장애로로부터 데이터베이스를 보호하고 서비스를 지속할 수 있게 해 주는 시스템이 존재한다.

### 이식성

Oracle Database 는 전부 C 언어로 쓰여있기 때문에 폭넓게 운용할 수 있다.

관리 및 개발의 효율성을 높이기 위한 다양한 툴을 이용할 수 있다.

### 그 밖의 특징

트랜잭션이 자동으로 시작된다 : 고성능 트랜잭션 처리를 제공해 주므로 속도가 빠르다.

DDL 이나 DCL 을 실행한 후, commit 이 자동으로 실행된다.

대규모 데이터베이스를 지원한다.

SQL 문을 실행하는 가장 효율적인 방법을 선택 → 비용 최소화를 위해 테이블과 인덱스를 분석한다.

## MySQL, PostgreSQL 과의 비교

MySQL 은 MySQL AB 가 개발한 RDBMS 이지만, 현재는 Oracle 사에서 관리 및 운영하고 있다.

PostgreSQL 은 PostgreSQL Global Development Group 에 의해 개발되고 있다.

### 라이센스의 차이

Oracle 은 고기능 및 고성능인만큼 압도적으로 가격이 비싸다

Enterprise Edition, Standard Edition 2 로 나뉘는데 가격이(...) Processor 라이센스라고 불림.

PostgreSQL 은 오픈소스 소프트웨어

MySQL 은 Community Edition 은 무료이고, Standard Edition, Enterprise Edition 이 있고 가격은 Oracle 의 1/10 정도. 연간 서브스크립션(1-4소켓 서버/년)

### 파생된 DB 의 종류

PostgreSQL 과 MySQL 으로부터 파생된 데이터베이스가 굉장히 많이 이용되고 있다.

- PostgreSQL : EDB Postgres, Powergres, Postgres Pro, Fujitsu Synfoware, Amazon Redshift, IBM Netezza, Pivotal Greenplum, hp Vertica : Netezza 랑 Vertica 는 기본 엔진으로 이걸 이용한다고.
- MySQL : MariaDB, Amazon Aurora

### 스토리지 아키텍쳐 면의 차이

Oracle 이나 MySQL 이 '갱신형'의 아키텍쳐인 반면, PostgreSQL 은 스토리지의 관리 방법이 '추기형'이기 때문에, 기존 데이터에 대해 덮어쓰기를 하는 것이 아닌 비어 있는 영역을 사용하여 갱신된 데이터를 추가로 기재하는 방식

- 갱신형 : 대상 레코드를 그대로 갱신
- 추기형 : 기존 레코드를 보존하면서 신규 레코드를 추가하여 참조할 곳을 갱신
    
    추가로, 데이터 사이즈 면에서는 버큠(vaccum) 기능에 의해 해결한다.
    

### 데이터베이스의 기능 비교

데이터 타입이나 명명규칙, 서식 등은 각자 상이한 형식을 사용하지만 큰 차이는 없다.

Oracle 은 체크 제약이 있지만, MySQL 은 트리거로 실행할 필요가 있다.

Oracle 은 시퀀스가 존재하는 반면, MySQL 은 존재하지 않는 대신 자동 인크리먼트로 정의한다.

Oracle 은 シノニム(오브젝트에 대한 별명을 정의) 기능이 존재하는 반면, MySQL 과 PostgreSQL 은 없다.

Oracle 은 NULL 과 빈 문자열은 동일하다고 판정하지만, MySQL 과 PostgreSQL 은 다른 개념으로 정의한다.

Oracle 은 트랜잭션이 임의의 단위지만, MySQL 과 PostgreSQL 은 기본적으로 SQL 문 단위가 확정되어 있다.

그 밖의 실행 계획의 차이나 힌트 구문의 유무 등

## 과제
좀 더 자세한 내용이 여기에 있다. 이 내용 숙지해서 추가해두기  
https://velog.io/@jisoo1170/Oracle-MySQL-PostgreSQL-%EC%B0%A8%EC%9D%B4%EC%A0%90%EC%9D%80

포스그레 병렬처리 내용도 추가해두기
