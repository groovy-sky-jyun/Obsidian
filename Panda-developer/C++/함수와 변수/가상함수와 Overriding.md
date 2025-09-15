### 가상 함수
`virtual` 키워드로 선언된 멤버 함수를 가상 함수라 한다.

virtual 키워드는 정적 바인딩과 달리 컴파일러에게 자신의 <span style="color:rgb(255, 207, 61)">호출 바인딩을 실행 시간까지 미루도록 지시</span>한다.

기본 클래스나 파생 클래스 어디에서나 선언될 수 있다.

```cpp title:가상함수
class Base{ 
public:
	virtual void f();
};
```




---
### Overriding
파생 클래스에서 기본 클래스의 <span style="color:rgb(255, 207, 61)">가상 함수를 재정의</span> 하는 것을 Overriding이라 한다.

이는 <span style="color:rgb(255, 207, 61)">실행 시간 다형성</span>을 실현한다.

#### 목적
파생 클래스들이 자신의 목적에 맞게 가상 함수를 재정의 하도록 하기 위함이다.

기본 클래스의 가상 함수는 파생 클래스에서 구현해야 할 함수 인터페이스를 제공하는 역할을 한다.

#### 사용 방법
<span style="color:rgb(255, 207, 61)">기본 클래스에서는 virtual 키워드</span>를 이용해 가상 함수를 작성한다.

이를 상속받은 <span style="color:rgb(255, 207, 61)">파생 클래스에서는</span> 함수에 virtual 키워드를 붙이지 않아도 가상 함수로 인식된다.

허나, 그러면 보는 사람 입장에서는 이 함수가 가상함수인지 파악하기가 어려워 <span style="color:rgb(255, 207, 61)">`override`라는 식별자</span>를 사용해준다.

컴파일러는 `override` 키워드를 보고 기본 클래스에 해당 가상 함수가 없거나, 함수 시그니처(이름, 매개변수, 타입)이 잘못 작성되었으면 오류를 발생시킨다.
```cpp title:overriding hl:7,12,19
class Shape {  
public:
	void paint(Shape* shape) {
		shape->draw();
	}

	virtual void draw() {}
};

class Circle : public Shape {
public:
	virtual void draw() override {
		cout << "circle" << endl;
	}
};

class Rect : public Shape {
public:
	virtual void draw() override {
		cout << "rect" << endl;
	}
};

int main() {
	Circle* circle = new Circle();
	Rect* rect = new Rect();
	Shape* shape = new Shape();
	shape->paint(circle); //출력: circle
	shape->paint(rect); //출력: rect
}
```

---
### 동적 바인딩
동적 바인딩은 실행 중 가상 함수가 호출되면, 객체 내 오버라이딩된 가상 함수를 동적으로 찾아 호출한다.

즉, 파생 클래스의 객체에 대해, 기본 클래스 포인터로 가상 함수가 호출될 때 일어난다.

이는 파생 클래스에서 재정의한 가상 함수의 호출을 보장받을 수 있다는 것을 뜻한다.
>**동적 바인딩이 발생하는 상황**
>- 기본 클래스의 멤버 함수에서 가상 함수 호출
>- 파생 클래스의 멤버 함수에서 가상 함수 호출
>- 외부 함수에서 기본 클래스의 포인터로 가상 함수 호출
>- 다른 클래스에서 가상 함수 호출

<span style="color:rgb(255, 207, 61)">실행 시간 바인딩</span>이라고도 부르며 정적 바인딩과 달리 포인터/참조 타입에 따라 실행될 함수가 결정되는것이 아닌 <span style="color:rgb(255, 207, 61)">객체 타입</span>에 따라 결정된다.
```cpp title:기본클래스멤버함수에서가상함수호출 hl:4,14,21
class Base { 
public: 
	void printStatus() {
		show(); // 가상 함수 호출
	}

	virtual void show() {
		cout << "base" << endl;
	}
}; 

class Derived : public Base {
public:
	virtual void show() override {
		cout << "derived" << endl;
	}
};

int main() {
	Base* pBase = new Derived();
	pBase->printStatus(); //출력: derived
}
```

```cpp title:외부함수에서가상함수호출 hl:3,10,16,17
class Base {  
public:
	virtual void show() {
		cout << "Base" << endl;
	} 
};

class Derived : public Base {
public:
	virtual void show() override {
		cout << "Derived" << endl;
	}
};

int main() {
	Base* ptr = new Derived();
	ptr->show(); //출력: Derived
}
```