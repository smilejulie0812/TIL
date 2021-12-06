서버에 초기 설정된 python 버전은 2.7.12 이며, 이를 update-alternatives 명령어로 버전 관리할 경우 타 python 기반 시스템에 이슈가 발생하게 된다.

이를 방지하기 위해, 사용자별로 python 버전 관리가 가능한 pyenv 를 설치하여 ElastAlert 설치 사전 작업을 수행한다.

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

/home/wmp-user/.bash_profile
```bash
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init --path)"
```

```bash
alias alertCode='cd /home/smileejulie/.pyenv/versions/3.6.15/lib/python3.6/site-packages/elastalert'
```

변경된 bashc 를 적용하기 위해 source 를 실행한다.
```bash
source ~/.bash_profile
Install python 3.6.15
```
아래의 명령어를 참고로 python 3.6.15 를 설치한다.

### pyenv 로 설치 가능한 리스트 출력
```bash
pyenv install --list
```
 
### python 3.6.15 설치
```bash
pyenv install 3.6.15
```
 
### 시스템에 설치된 파이썬 버전 확인
```bash
pyenv versions
python --versions
``` 
 
이것으로 특정 사용자에서 기본값으로 사용할 python 3.6.15 가 설치되었다(슈퍼유저 포함한 타 사용자에는 영향 없음).
