### Static 변수/함수
Static은 같은 클래스에 속한 모든 인스턴스가 공유하여 사용해야 하는 변수/함수를 지정할 때 사용한다.

main이 실행되기 전 한번만 초기화 되며, Stack이나 Heap영역이 아닌 Data Segment에 할당된다.

다음은 static의 위치에 따른 특징을 살펴볼 것이다.

### 클래스 내부 static
<span style="color:rgb(188, 149, 218)">클래스 내부에서 선언</span>된 static 멤버 변수/함수는 반드시 <span style="color:rgb(188, 149, 218)">클래스 외부에서 정의</span>를 해주어야 한다. 클래스 내부에서 선언되었지만 객체에 속하지 않기 때문이다.
```cpp
class Car{
public:
	static int num;
	static void print();
};

//클래스 외부에서 초기화
int Car::num = 5;
void Car::print(){
	cout << num << endl; 
}

int main(void) { ... }
```

##### static 멤버 변수
static 멤버 변수가 public으로 선언되어 있는 경우 :
객체를 생성하지 않고도 접근할 수 있다. static 멤버 변수를 같은 클래스의 여러 인스턴스가 공유한다고 생각하면 쉽다. 그래서 처음 변수 초기화는 한번 진행되지만 여러 인스턴스에서 값을 수정할때마다 

클래스 이름이 Car이고, 멤버 변수 이름이 num 이라면 다음과 같이 접근할 수 있다.
`Car::num`


static 멤버변수가 public으로 선언되어 있다면 객체가 생성되지 않아도 접근이 가능하다. 
객체에 귀속되지 않고 클래스 자체에 속한 것으로 모든 인스턴스가 공유하는 하나의 변수/함수이다.
그렇기에 객체 없이도 접근이 가능하다. 
`ClassName::VariableName`
클래스 내부에 선언할 경우 외부에서 반드시 정의를 해주어야 한다. 
초기화는 한번만 진행할 수 있으며, 생성자가 아닌 클래스 외부에서 초기화를 해야한다.

```cpp
#include <iostream>
using namespace std;

class Car{
public:
	static int num;
	
	string name = "Morning";
	
	static void print();
};

//클래스 외부에서 초기화
int Car::num = 5;
void Car::print(){
	num++;
	cout << num << endl; //name은 접근 불가능
}
int main(void) {
	//Car 객체 생성 없이 num 접근
	cout << Car::num << endl; //출력: 5
	Car::print(); //출력: 6
}
```
이를 통해 static 멤버 변수는 객체 내부에 위치하지 않는것을 알 수 있다.
static 함수에서는 클래스의 멤버 변수에 접근할 수 없는것을 볼 수 있는데
이는 static 함수는 객체에 속해있지 않기에 this 포인터를 가지고 있지 않기 때문이다.
static 함수 내에서 클래스 멤버 변수에 접근하려면 함수 내에서 객체를 생성해주어야 한다.
```cpp
void Car::print()
{
	Car car;
	cout << car.name << endl; //출력: Morning
}
```
객체간 값을 공유하고자 할 때 사용할 수 있다.
#### Static 멤버 함수
Static 멤버 함수는 객체 멤버가 아니다. 그렇기에 객체에 private으로 선언된 변수에는 접근할 수가 없다. 

### 함수 내부 static (지역 변수)
```cpp
class Car{
public:
	void func(){
		static int count = 0;
		count++;
		std::cout << count << std::endl;
	}
}
int main(void) {
	Car* car_1 = new Car();
	car_1->func(); //출력: 1

	Car* car_2 = new Car();
	car_2->func(); //출력: 2
}
```

함수가 호출될때 한번만 초기화된다.
함수 종료후에도 값을 유지하고 있다. (다음 호출 시 이전 값 사용, 프로그램 종료 전까지 계속 유지)
Car 클래스에서 파생된 car_1, car_2 객체는 count 변수를 공유하고 있다.
함수에서 사용되는 변수의 값을 한번만 초기화 하고 싶을 때 사용할 수 있다.

### 클래스 외부(파일 스코프) static (전역 변수/함수)
```cpp
static int callednum = 1;
static void printcallednum() {
	callednum++;
	std::cout << callednum << std::endl;
}

int main(void){
	printcallednum(); //출력: 2
	printcallednum(); //출력: 3
}
```
이렇게보면 그냥 일반 전역변수/함수와 다른 특별한 차이가 없어보인다.

하지만 분명한 차이가 있다.
일반 전역변수/함수는 external이 가능하다. 다른 .cpp 파일에서 extern으로 호출이 가능하지만, static으로 선언된 전역변수/함수는 다른 .cpp 파일에서 호출이 불가능하다.

즉, static 전역 변수/함수는 선언된 파일 안에서만 사용이 가능하다.
이는 캡슐화를 위해 외부에서 접근할 수 없도록 할 때 사용할 수 있다.