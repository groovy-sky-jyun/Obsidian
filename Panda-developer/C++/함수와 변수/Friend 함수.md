### 목적
클래스 멤버 함수로 적합하지 않아서 외부에 작성하지만, 특정 클래스의 private, protected 멤버에 접근해야 하는 경우 사용된다.

해당 클래스 멤버가 아닌 함수에게 friend 선언을 해줌으로써 해당 클래스 멤버와 동일한 접근 권한을 부여한다. 

그럼 해당 클래스 멤버가 아님에도 해당 클래스의 private, protected 함수/변수에 접근이 가능해진다.

```cpp title:friend예시 hl:3-5,10
class Wallet; //전방선언
 
int sum(Wallet a, Wallet b){
	return a.money + b.money;
} 

class Wallet{
int money;
public:
	friend sum(Wallet a, Wallet b);
};
```


### 특징
- 상속에 영향을 미치지 않는다. (<span style="color:rgb(255, 207, 61)">선언된 클래스에만 접근 권한이 허용</span>된다.)
	- 부모 클래스의 friend라고 해서 자식 클래스의 private에 접근할 수 없다.
	- 자식 클래스의 friend라고 해서 부모 클래스의 public, protected, private에 접근할 수 없다.
- 접근 지정자 영향을 미치지 않는다.
	- 선언 위치와 상관없이 동일한 접근 권한을 부여한다.
- <span style="color:rgb(255, 207, 61)">개수 상관없이</span> friend 함수 선언을 할 수 있다.
- <span style="color:rgb(255, 207, 61)">함수 및 클래스</span>를 friend로 선언할 수 있고, 변수는 불가능하다.
- 주로 <span style="color:rgb(255, 207, 61)">연산자 함수</span>를 friend 함수로 선언한다.

### 선언 방법
##### <mark style="background: #D2B3FFA6;">전역 함수를 friend함수로 선언</mark>
전역 함수를 클래스 내부에 friend 키워드로 선언할 때 <span style="color:rgb(255, 207, 61)">접근 지정자와 상관없이 아무 곳에나 작성</span>하면 된다.

friend 함수는 해당 클래스 멤버에 접근할 권한을 가진것 뿐이지 실상은 <span style="color:rgb(255, 207, 61)">전역함수</span>이기 때문이다. 

아래의 예시처럼 전역함수 sum을 private에 friend함수로 작성한다 하더라도 어디에서든 sum함수를 호출할 수 있다.
```cpp title:friend함수선언위치 hl:4,13-15
class Wallet {  
private: 
	int money;
	friend int sum(Wallet a, Wallet b);

public:
	Wallet(int m) {
		money = m;
	}
};

//전역함수
int sum(Wallet a, Wallet b) {
	return a.money + b.money;
}

int main() {
	Wallet w1(5), w2(10);
	cout << sum(w1, w2) << endl;
	return 0;
}
```

하나의 전역변수를 여러 클래스에서 friend함수로 선언하고 함께 사용할 수 있다.
```cpp title:여러클래스에공통friend함수선언 hl:4-9,11-16,18-20
class Car;      
class Driver;

class Car {
	int car_id;
public:
	Car(int id) : car_id(id) {}
	friend void info(Car& car, Driver& driver);
};

class Driver {
	int driver_id;
public:
	Driver(int id) : driver_id(id) {}
	friend void info(Car& car, Driver& driver);
};

void info(Car& car, Driver& driver) {
	cout << car.car_id << " car has " << driver.driver_id << endl;
}

int main() {
	Car car(1);
	Driver driver(777);
	info(car, driver);

	return 0;
}
```


##### <mark style="background: #D2B3FFA6;">다른 클래스 멤버 함수를 friend함수로 선언</mark>
전역함수가 아닌 다른 클래스 멤버 함수를 friend로 선언하려면 다음과 같이 해야한다.

`friend 반환타입 클래스명::함수이름();`

다른 클래스 멤버 함수가 private이든 protected이든 상관없이 friend 함수로 선언할 수 있다. (접근하는 주체가 다른 클래스 멤버 함수이기 때문)

```cpp title:다른클래스멤버함수friend선언 hl:3-6,12
class Car;   

class RentalCompany {
public: //private,protected여도 friend로 선언 가능
	int CarNum(Car& car);
};

class Car {
	int car_id;
public:
	Car(int id) : car_id(id) {}
	friend int RentalCompany::CarNum(Car& car);
};

int RentalCompany::CarNum(Car& car) {
	return car.car_id;
}

int main() {
	Car car(777);
	RentalCompany rental;

	cout << rental.CarNum(car) << endl;

	return 0;
}
```

##### <mark style="background: #D2B3FFA6;">다른 클래스 멤버 함수 전체를 한번에 friend함수로 선언</mark>
이는 되게 간단하다.
모든 멤버 함수를 friend함수로 선언할 <span style="color:rgb(255, 207, 61)">클래스를 friend로 선언</span>하면 된다. 
```cpp
class Car{
	//...
	friend RentalManager;
}
```