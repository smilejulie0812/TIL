## find 명령어로 여러 파일의 문자열을 자유롭게 관리하기
인프라 업무란... 명령어를 얼마나 아느냐에 따라 노가다(...)의 레벨이 달라지는 것...

### 여러 파일의 특정 문자열 한번에 바꾸기
```bash
find ./ -name "*.txt" -exec sed -i 's/<바꾸기 전 문자열>/<바꾸고자 하는 문자열>/g' {} \;
##### 물론 sed 명령어만 사용해도 가능하다
sed -i 's/<바꾸기 전 문자열>/<바꾸고자 하는 문자열>/g' *.txt
```
 
### 여러 파일의 특정 문자열을 한번에 삭제
```bash
find ./ -name "*.txt" -exec sed -i '<삭제할 문자열>/d' {} \;
```

### 해당 문자열이 포함된 줄만 삭제하지 않고 나머지 내용 삭제
```bash
find ./ -name "*.txt" -exec sed -i '<삭제할 문자열>/!d' {} \;
```
 
### 여러 파일의 특정 문자열을 특정 라인에 한번에 추가하기
```bash
find . -name "*.txt" -exec perl -pi -e '$.==<문자열 넣고자 하는 라인 수> and print "<추가하고자 하는 문자열>"' {} \;
##### 예시 : find . -name "*.yaml" -exec perl -pi -e '$.==1 and print "\napple\n"' {} \;
```
 
#### find 명령어 옵션
* -exec : find 로 찾은 결과 대상에 대해 다음의 명령어를 적용시킨다
* {} : find 에서 찾아낸 결과(=.txt 로 끝나는 모든 파일명들) 가 하나씩 들어가는 부분( for 문같은 형식으로 들어가나...?)
* \; : -exec 다음에 쓰인 명령어( perl -pi -e '$.==1 and print "\napple\n"' ) 를 실행하는 부분

### perl 명령어 옵션
* -e : 주어진 perl 명령을 실행한다
* -p : 지정한 파일을 대상으로 작업한다
* -i : 원본파일을 결과파일로 대체한다
