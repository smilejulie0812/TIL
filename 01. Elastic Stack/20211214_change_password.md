## 패스워드 변경하기
진짜 별 거 아닌 내용이긴 한데, 처음으로 해 봐서 적어둔다.  
API 로 유저 패스워드 변경하는 방법.  
```
POST /_security/user/<유저명>/_password
{
  "password" : "<패스워드>"
}
```
해당 API 로 elastic(admin) 유저 패스워드도 변경 가능하다.

### 참고
https://www.elastic.co/guide/en/elasticsearch/reference/current/security-api-change-password.html
