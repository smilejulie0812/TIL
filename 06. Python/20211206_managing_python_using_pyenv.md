## pyenv 로 유저별 python 버전 관리하기

### 발단
최신버전 ElastAlert 는 python 3.6 이상에서만 작동한다길래 update-alternatives 명령어로 버전 관리하려던 멍청한 시스템 엔지니어 덕분에 인프라팀에 VM 재설치 요구까지 했다(...)  
(그 멍청한 시스템 엔지니어가 바로 나에요 나!! 무슨 JAVA 도 아니고...)  

이와 같은 이슈를 방지하기 위해, 사용자별로 python 버전 관리가 가능한 pyenv 를 설치하여 ElastAlert 설치 사전 작업을 수행하는 방법을 익혔다.

### Install Relative Packages
pyenv 를 설치할 때 의존성 문제가 발생할 가능성을 제거하기 위해 아래의 명령어로 관련 패키지를 사전 설치한다.
```bash
sudo apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev \
libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev \
xz-utils tk-dev
Install pyenv
```

git clone 하여 pyenv 를 설치한다.
```bash
git clone https://github.com/pyenv/pyenv.git ~/.pyenv
Set .bash_profile
```

해당 사용자가 사용할 쉘(bash) 설정파일에 아래 내용을 추가한다.
```bash
cd /home/smileejulie/.bash_profile
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init --path)"
```

변경된 bashc 를 적용하기 위해 source 를 실행한다.
```bash
source ~/.bash_profile
Install python 3.6.15
```

아래의 명령어를 참고로 python 3.6.15 를 설치한다.
```bash
### pyenv 로 설치 가능한 리스트 출력
pyenv install --list
### python 3.6.15 설치
pyenv install 3.6.15
### 시스템에 설치된 파이썬 버전 확인
pyenv versions
python --versions
``` 

이건 덤. 코드 디렉토리가 너무 깊이 숨어 있는걸 alias 로 간편하게 사용할 수 있게 할 수 있다.
```bash
alias alertCode='cd /home/smileejulie/.pyenv/versions/3.6.15/lib/python3.6/site-packages/elastalert'
```

이것으로 smileejulie 사용자 전용으로 사용할 python 3.6.15 가 설치되었다(슈퍼유저 포함한 타 사용자에는 영향 없음).
