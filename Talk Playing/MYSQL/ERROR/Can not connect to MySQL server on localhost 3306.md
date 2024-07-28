<br>

## error

cmd 창에서 `mysql -u root -p > 비밀번호` 를 입력했을 때 
> Can't connect to MySQL server on 'localhost:3306'

라는 오류 발생

<br>

### 해결 방법 

mysql이 실행이 안된 상태여서 난 오류이기 때문에 #MySqlWorkBench 를 실행하거나
> ##### 서비스 > MYSQL 우클릭 > 시작
> 후에 다시 cmd 창을 켜서 로그인을 하면 로그인이 된다.
> 
![[Pasted image 20240728155000.png]]