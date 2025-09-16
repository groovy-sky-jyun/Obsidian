### 가상 함수
가상 함수는 다른 클래스에서 <span style="color:rgb(255, 207, 61)">Overriding</span> 할 수 있는 권한이 부여된 함수이다.

즉, <span style="color:rgb(255, 207, 61)">동적 바인딩</span>을 가능하게 하여 파생 클래스에서 <span style="color:rgb(255, 207, 61)">재정의</span>될 수 있도록 설계된 멤버 함수이다. <span style="color:rgb(125, 125, 125)">(다형성)</span>

기본 클래스의 가상 함수는 파생 클래스에서 구현해야 할 <span style="color:rgb(255, 207, 61)">함수 인터페이스</span>를 제공하는 역할<span style="color:rgb(125, 125, 125)">(추상화)</span>을 한다.

#### 선언 방법
어느 클래스 멤버 함수에서나 선언될 수 있다.
`virtual 반환타입 함수명(매개변수)`

```cpp title:가상함수
class Base{ 
public:
	virtual void f();
};
```

#### 특징
가상함수의 <span style="color:rgb(255, 207, 61)">virtual 속성은 상속</span>되는 성질을 지니고 있다.

파생 클래스에서 가상 함수를 Overriding 할 때 <span style="color:rgb(255, 207, 61)">virtual 키워드를 생략해도 가상 함수로 인식</span>된다.
<span style="color:rgb(125, 125, 125)">(Overriding을 명시하기 위해 virtual은 생략하지 말자)</span>

>**virtual 생략 시 이름 숨김과 같지 않나?**
> virtual을 생략하면 이름 숨김과 같아 보일 수 있지만 virtual 속성은 상속되므로 여전히 가상함수를 나타낸다.

```cpp
class Base{ 
public:
	virtual void f();
};

class Derived{
public:
	void f(); //이름숨김x 가상함수o
};
```

#### 접근 지정자
가상 함수 또한 보통 함수와 마찬가지로 public, protected, private 중 자유롭게 사용할 수 있다.

---
### Overriding
가상 함수를 파생 클래스에서 재정의 하는 것을 Overriding 이라고 한다.

함수 재정의를 통해 파생 클래스는 자신의 목적에 맞게 함수 내용을 수정할 수 있다.

#### 사용 방법
`virtual 반환타입 함수명(매개변수) override;`

가상 함수가 있는 클래스에서 파생된 클래스는 함수 시그니처(이름, 매개변수, 타입)를 동일하게 작성함으로써 Overriding 할 수 있다.

해당 함수가 기본 클래스의 가상함수가 아닌 재정의된 가상 함수라는것을 명시적으로 알려주기 위해 `virtual`와 `override` 키워드를 붙여준다. <span style="color:rgb(125, 125, 125)">(생략 가능하지만 명시해주자.)</span>

>**`override` 키워드의 역할**
>override는 재정의된 함수라는 것을 명시하기 위함도 있지만 다른 역할도 있다.
>
>컴파일러는 `override` 키워드를 보고 기본 클래스에 해당 가상 함수가 없거나, 함수 시그니처(이름, 매개변수, 타입)이 잘못 작성되었으면 오류를 발생시킨다.
>
>이를 통해 개발자는 어디에 오류가 발생한 것인지 쉽게 알 수 있다.

<br>

#### 작동 원리
Overriding은 가상 함수를 재정의하는 것을 뜻하고, 오버라이딩 된 함수가 호출되는 원리는 동적 바인딩을 따른다.
[[정적 바인딩과 동적 바인딩]]

#### 사용 예시
- `Shape`에서는 `draw()`를 가상 함수로 선언하고 있다.
- `Circle`, `Rect`에서는 `draw()`를 Overriding하고 있다.
- 동적 바인딩에 의해 `shape->draw()`는 실행 시간에 포인터`shape`이 실제로 가리키는 객체 `Circle`와 `Rect`에 재정의된 `draw()`함수를 각각 호출한다.
```cpp title:overriding hl:4,6,11,18,27,28
class Shape {    
public: 
	void paint(Shape* shape) {
		shape->draw();
	}
	virtual void draw() {} //가상 함수
};

class Circle : public Shape {
public:
	virtual void draw() override { //overriding
		cout << "circle" << endl;
	}
};

class Rect : public Shape {
public:
	virtual void draw() override { //overriding
		cout << "rect" << endl;
	}
};

int main() {
	Circle* circle = new Circle;
	Rect* rect = new Rect
	Shape* shape = new Shape;
	shape->paint(circle); //출력: circle
	shape->paint(rect); //출력: rect
	
	delete circle;
	delete rect;
	delete shape;
}
```
