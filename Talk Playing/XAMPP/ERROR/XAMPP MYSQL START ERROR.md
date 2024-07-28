<br>

## error

XAMPP 에서 MySQL START 버튼을 눌렀을 때 실행이 안되는 경우 



<br>

### 해결 방법 
PORT번호가 앞전에 사용한게 제대로 종료되지 않아서 발생하는 오류일 수 있다. 그럴때는 이전에 사용한 PORT 번호를 종료시켜준다.
1. CMD 창을 관리자 권한으로 실행
2. 해당 포트번호로 실행되는 PID번호 검색
> ```netstat -ano | findstr 포트번호 ```
3. 프로그램 KILL
> ``` taskkill /f /pid 해당PID번호 ```