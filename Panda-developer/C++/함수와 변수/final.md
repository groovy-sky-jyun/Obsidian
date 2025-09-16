`final` 지시어를 사용해 1. <span style="color:rgb(255, 207, 61)">오버라이딩을 금지</span> 하거나, 2. <span style="color:rgb(255, 207, 61)">클래스 상속을 금지</span>할 수 있다.

---
### 1. 오버라이딩 금지 선언
설계자가 특정 가상 함수가 상속 계층의 특정 지점까지만 재정의 되기를 원할 때 사용한다.

final 지시어를 사용한 특정 상속 계층(클래스 가상 함수) 이후로는 가상 함수를 오버라이딩 할 수 없다.

#### 선언 방법
가상 함수에만 사용 가능하다.
`virtual 반환타입 함수이름(매개변수) final;`

#### 사용 예시
아래의 코드를 보면 final 지시어를 사용해 `Rect` 클래스까지만 함수 오버라이딩이 설계하였다.

`RouncRect`에서 오버라이딩을 하는것은 잘못된 선언이므로 오류가 발생한다. 

Rect를 상속받은 클래스들은 Rect의 draw()를 그대로 사용해야 한다.

```cpp title:final오버라이딩금지 hl:8,12
class Shape{   
public:
	virtual void draw(); 
};

class Rect : public Shape{
public:
	virtual void draw() final; //overriding 금지
};

class RoundRect : public Rect{
	virtual void draw() override;//오류
};
```

---
### 2. 상속 금지 선언
설계자가 상속 계층을 마무리하고 싶을 때 사용한다.

final 지시어를 사용한 클래스는 더 이상 파생 클래스를 가질 수 없다.

#### 선언 방법
모든 클래스에 사용 가능하다.
`class 클래스명 final {}`
`class 클래스명1 final : public 클래스명2 {}`

#### 사용 예시
아래의 코드를 보면 rect 클래스에서 final 지시어가 사용되었으므로 다른 클래스들은 Rect 클래스를 상속할 수 없다.

그러므로 `Rect`를 상속받고 있는 `RoundRect` 클래스는 오류가 발생하는 것이다.
```cpp title:final상속금지 hl:4,7
class Shape {  
	//... 
};
class Rect final : public Shape{ //상속 금지
	//...
};
class RoundRect : public Rect{ //오류
	//...
};
```