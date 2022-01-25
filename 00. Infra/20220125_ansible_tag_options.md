# Ansible Tag Option
### Q. Role 내부의 특정 task 파일을 지정해서 playbook 실행 가능한지?

A. 불가능하다! 대신, 각 task 마다 **tag** 를 붙여서 해당 tag 를 기준으로 playbook 을 실행할 수 있다

## Tag

크고 복잡한 Playbook 을 사용할 때, 특정 Task 에 대해서만 playbook 을 사용할 때 **Tags** 를 사용하면 편리

1. 각각의 Task 에 Tag 를 추가
2. Playbook 을 실행할 때 Tag 를 선택 또는 스킵할 옵션 추가

### 관련 옵션

```bash
## Tag 와 상관없이 모든 Task 를 실행 : --tags 옵션의 기본값
ansible-playbook <playbook 파일> --tags all
## 해당 Tag 가 달린 Task 를 실행
ansible-playbook <playbook 파일> --tags <Tag 1>,<Tag 2>
## 해당 Tag 가 달린 Task 외 모두 실행
ansible-playbook <playbook 파일> --skip-tags <Tag 1>, t<Tag 2>
## Tag 가 있는 Task 만 실행
ansible-playbook <playbook 파일> --tags tagged
## Tag 가 없는 Task 만 실행
ansible-playbook <playbook 파일> --tags untagged
```

> —tags all 옵션은 never 태그를 어떻게 처리하나? 태그를 무시한다면 never 태그도 실행할텐데, 그럼 never 태그를 붙이는 의미가 없는 것 같고... 테스트해보기
> 

### 예시

그림: excalidraw 로 현재 상태 그려두기

### 스페셜 옵션

- **always** : 어떤 Tag 를 지정하더라도 해당 Tag 를 달아둔 Task 는 항상 실행
    - 모든 Task 는 특정 Tag 를 지정하지 않는 한 기본값이 always 임
    - `--skip-tags always` : always Tag 가 달린 Task 는 실행하지 않음
    
    ```yaml
    ## 다른 Tag 가 있던 없던 해당 Task 는 반드시 실행된다
    tasks:
    - name: Print a message
      ansible.builtin.debug:
        msg: "Always runs"
      tags:
      - always
    ```
    
- **never** : 어떤 Tag 를 지정하더라도 혹은 Tag 를 지정하지 않더라도 해당 Tag 를 달아둔 Task 는 실행 안함
    - tag 옵션 없이 playbook 사용시 해당 tag 가 달린 task 는 스킵
    - **상위 tag 로 playbook 실행시 그대로 실행됨**
    - `--tags never` : never Tag 가 달린 Task 를 실행함
    
    ```yaml
    ## debug Tags 를 옵션에 넣지 않는 한 해당 Task 는 실행되지 않는다
    tasks:
      - name: Run the rarely-used debug task
        ansible.builtin.debug:
         msg: '{{ showmevar }}'
        tags: never,debug
    ```
