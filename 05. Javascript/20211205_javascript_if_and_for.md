## Javascript 로 코딩테스트 문제 풀이를 시작하자
회사에서 내려온 많은 지령 중에 Node.js 를 사용할 수 있도록 익히라는 과제가 내려왔고, 그걸 위해 Javascript 부터 익히고자 한다.  
그러기 위해서는 코딩 스터디에서 매주 과제로 주어지는 문제를 Javascript 로 풀면서 공부할 필요가 있다(python 공부는 도대체 언제 끝날까...)  
쌩기초를 다지기 위해 두 가지 쓰임만 간단히 써놓고자 한다.
### if 문
```javascript
### 입력된 숫자를 변수 var_num 로 선언
var var_num = prompt( 'Enter Number', '' );
### var_num 이 100 보다 작을 때
if ( var_num < 100 ) {
  document.write ( '<p>Your number is less than 100</p>' );
### var_num 이 100 과 같을 때
} else if ( var_num == 100 ) {
  document.write ( '<p>Your number is 100</p>' );
### var_num 이 100 보다 클 때
} else {
  document.write ( '<p>Your number is greater than 100</p>' );
}
```
### for 문
#### 일반 for 문
```javascript
for (var i = 1; i < 10; i++) {
  document.write(i + "<br>");
}
```
#### for in 문
```javascript
var obj = { name : "사과", price : 1500 };

for (var i in obj) {
  document.write(i + "<br>");
}
```
#### for of 문
```javascript
var arr = [1, 2, 3, 4, 5];

for (var i of arr) {
  document.write(i + " ");
}
```
