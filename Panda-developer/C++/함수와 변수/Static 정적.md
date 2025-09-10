### static 멤버
- 한 클래스에 객체가 여러개 생성되었더라도 static멤버는 <span style="color:rgb(255, 207, 61)">하나만 생성</span>된다.
- 해당 클래스의 모든 객체가 공유하는 <span style="color:rgb(255, 207, 61)">공유 자원</span>이다.
- <span style="color:rgb(255, 207, 61)">객체 생성 없이도 접근</span>할 수 있다.
- <span style="color:rgb(255, 207, 61)">클래스 멤버</span>라고 한다.

### 생명 주기
**생성**
- 프로그램 시작 시 생성
- 객체가 생성되기 전에 생성

**소멸**
- 프로그램 종료 시 소멸
- 객체가 사라져도 프로그램이 실행 중이라면 소멸x

---
### 메모리 위치
프로그램의 전체 생명 주기를 함께해야 하므로 <span style="color:rgb(255, 207, 61)">데이터 영역</span>에 할당된다.

Stack의 경우 함수 호출이 끝나면 메모리가 자동으로 해제되고, Heap의 경우 동적 할당된 메모리가 저장되는 영역이므로 해당되지 않는다.

### 선언과 정의
- static 멤버는 객체가 생성되기 전부터 초기화가 되어 있어야 하므로 <span style="color:rgb(255, 207, 61)">전역 범위에서 정의 및 초기화</span> 작업이 필요하다.
- 정의 : `타입 클래스명::static 멤버 = 초기값`
- 캡슐화를 위해 가급적 전역변수/함수 대신 클래스에서 static멤버로 선언하자.
```cpp hl:6
class Person{
public:
	static int sharedMoney;
} 

int Person::sharedMoney = 10; //제거하면 오류
int main(){
	cout << Person::sharedMoney << endl; // 10
	Person p;
	cout << p.sharedMoney << endl; // 10
}
```


>##### 초기화 작업 시 주의할 점
>1. <span style="color:rgb(221, 186, 248)">**클래스 내에서 선언과 동시에 정의**</span>를 해주면 오류가 발생한다.
>	- `static int sharedMoney = 10; //오류`
>	- 클래스 내부에서 static 멤버를 정의하고 초기화하면, 해당 클래스를 포함하는 모든 파일이 각자 메모리 공간을 할당하려 시도한다.
>	- 링커는 하나만 정의되어야 하는 값이 여러곳에서 정의되었다고 판단하고 <span style="color:rgb(221, 186, 248)">다중 정의 오류</span>를 발생시킨다.
>	- 그래서 static 멤버는 반드시 클래스 외부의 전역 범위에서 정의해야하는 규칙이 있는 것이다.
>2. 여러 소스 파일에서 <span style="color:rgb(221, 186, 248)">**초기화를 중복 작성**</span>할 경우 오류가 발생한다.
>	- `int Person::sharedMoney = 10;`을 여러 파일에 중복 작성 시, 링커는 어떤 값으로 초기화를 해야할 지 모른다.
>	- 이는 <span style="color:rgb(221, 186, 248)">유일 정의 규칙(ODR)을 위반</span>했으므로 오류가 발생한다.
---
### 접근 지정자에 따른 접근 권한
하나의 멤버를 여러 객체가 공유한다 뿐이지 접근 지정자에 따른 접근 권한이 달라지지는 않는다.

**public**
- 클래스 외부의 누구든 접근 가능하다.
- `클래스명::변수명`, `클래스명::함수명()`로 객체가 생성되기 전에 접근할 수 있다.
- 값을 공유하고 변경할 수 있다.

**protected**
- 해당 클래스와 그 클래스를 상속받은 클래스들만 접근 

**private**
- 해당 클래스의 멤버 함수만 접근 가능

### static 멤버 함수의 접근 권한
**1. static 멤버 변수/함수만 사용할 수 있다.**
non-static 변수/함수가 생성되기 전에 실행될 수 있어야 한다. 
```cpp hl:4
  class Person{
  int money;
  public: 
	  static int getMoney() { return money; } //오류
  }
  ```

**2. `this 포인터 변수`를 사용할 수 없다.**
객체가 생성되기 전에 실행될 수 있어야 한다.
```cpp hl:5
  class Person{
  int money;
  public: 
	  static void addMoney(int n){
		  this->money += n; //오류
	  }
  }
```

**3. non-static 멤버 함수에서 static 멤버 접근은 제한이 없다.**
non-static 멤버 함수에서 static 멤버 함수/변수에 접근하는 것은 아무 문제가 되지 않는다.

---
### non-static
- 클래스 내에서 작성되는 non-static 변수/함수의 경우 객체와 생명주기를 함께한다.
- <span style="color:rgb(255, 207, 61)">인스턴스</span>라고 한다.