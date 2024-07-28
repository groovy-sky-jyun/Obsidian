<br>

## 사전 확인
XAMPP > MYSQL > Config > my.ini
- port 번호 확인
- password 확인
- `lower_case_table_names = 1` 추가
 >###### lower_case_table_names=0
 >테이블 이름이 저장된 그대로 유지되고, 대소문자를 구분한다. 
 >주로 Unix 계열 시스템에서 사용
 ><br>
> ###### lower_case_table_names=1
> 테이블 이름이 소문자로 변환되어 저장되고, 대소문자를 구분하지 않는다. 
> 주로 Windows에서 사용
> <br>
> ###### lower_case_table_names=2
> 테이블 이름이 저장된 그대로 유지되지만, 검색 시 소문자로 변환된다. 
> 주로 Unix 계열 시스템에서 사용

![[Pasted image 20240728172354.png|400]]
![[Pasted image 20240728172450.png|400]]

---
<br>

## 새로운 Connection 생성

##### 1. XAMPP 를 열어 MySQL을 START 해줍니다.
 ![[Pasted image 20240728165347.png|450]]
<br>

 ##### 2. MySQL Workbench를 열어 새로운 Connection을 생성해줍니다.
<br>

 ##### 3. 이때 xampp의 port 번호를 충돌문제로 인해 3308로 변경해주었기 때문에 새로 생성하는 Connection의 port도 3308로 변경해줍니다. 그 뒤 Test Connection을 통해 연결을 확인합니다.
 ![[Pasted image 20240728165446.png|450]]
 <br>

##  연동 확인
PHP로 연동을 확인해주기 위해 MySQL에서 TEST용 DATABASE를 만들고 값이 맞게 출력되는지 PhpMyAdmin으로 확인할 것이다.

우선 MySQL Workbench에서 해당 코드로 datebae를 만들어 user를 생성한다.
> ``` sql 
> CREATE DATABASE testdb; 
> 
> USE testdb;
> 
> CREATE TABLE users ( id INT AUTO_INCREMENT PRIMARY KEY, username VARCHAR(50) NOT NULL, email VARCHAR(100) NOT NULL, created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ); 
> 
> INSERT INTO users (username, email) VALUES ('testuser', 'test@example.com'); 
> ```

그런 뒤 `\xampp\htdocs` 폴더에 들어가서 `testdb.php` 파일을 생성해준다.
> ```php
>  <?php
$servername = "localhost";
$username = "root";
$password = "1234"; // root 계정의 비밀번호를 입력하세요
$dbname = "testdb";
$port = 3308; //변경한 포트번호
// MySQL 연결 생성
$conn = new mysqli($servername, $username, $password, $dbname, $port);
// 연결 확인
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}
//출력할 sql문
$sql = "SELECT id, username, email, created_at FROM users";
$result = $conn->query($sql);
if ($result->num_rows > 0) {
    // 출력 데이터 각 행
    while($row = $result->fetch_assoc()) {
        echo "id: " . $row["id"]. " - Name: " . $row["username"]. " - Email: " . $row["email"]. " - Created at: " . $row["created_at"]. "<br>";
    }
} else {
    echo "0 results";
}
$conn->close();
?> 
>```

인터넷에 `http://localhost/testdb.php` 주소를 들어가서 결과를 확인한다.
연동이 잘 되었다면 아래와 같이 MySQL Workbench로 입력한 데이터가 잘 출력되는것을 확인할 수 있다.
![[Pasted image 20240728172210.png|450]]