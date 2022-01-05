# .bash_profile 에서 터미널 색 설정하기
내 전용 유저에서만 쓰는 pyenv 설정이랑 alias 를 설정해 두려고 .bash_profile 을 만들어서 실행시켰다가 원래 적용되어 있었던 컬러링 설정이랑 ll alias 가 싹 날아가서 급 당황한 시스템 엔지니어의 간단한 반성문

각설하고 아래 내용을 추가해서 source 명령어로 돌리면 된다.
```bash
# ~/.profile
### 기본 쉘 타입이 bash 면 잔말말고 .bashrc 부터 적용하라는 의미
if [ "$BASH" ]; then
  if [ -f ~/.bashrc ]; then
    . ~/.bashrc
  fi
fi

mesg n || true

### 여기서부터가 중요하다 ###
### 기본 편집기를 vi 로 한 editor 를 사용하겠다는 의미.
### 이 설정으로 인해 ls 결과 디렉토리가 파랗게 표시되거나 vi 편집기에 색상이 표시된다
export EDITOR=vi
### 아래에서 쓸 쉘변수 PS1 을 설정할 때 사용할 변수 HOST_NAME 선언 : 보통 호스트명이 도메인값일 경우 도메인 정보를 제외한 내용만 가져오고자 하려는 것으로 보인다
HOST_NAME=`cat /proc/sys/kernel/hostname  |sed 's/.julielee.com//g'`
### PS1 지정 : 이걸로 <유저명>@<호스트명>:<절대경로> 의 출력 스타일을 지정한다
export PS1="\[\033[38;5;29m\]\u\[$(tput sgr0)\]\[\033[38;5;231m\]@\[$(tput sgr0)\]\[\033[38;5;208m\]$HOST_NAME\[$(tput sgr0)\]\[\033[38;5;15m\]:\[$(tput sgr0)\]\[\033[38;5;229m\]\w\[$(tput sgr0)\]\[\033[38;5;15m\]\n\[$(tput sgr0)\]\[\033[38;5;231m\]\\$\[$(tput sgr0)\] "
```

참고로, 특정 유저로 로그인했을 때 실행되는 설정 파일의 실행 순서는 아래와 같다.
<div class="notice" markdown="1">
/etc/profile -> /etc/bashrc -> ~/.bashrc -> ./bash_profile -> ~/bash_logout
</div>

### 참고
* https://dohk.tistory.com/191
* https://webdir.tistory.com/105
