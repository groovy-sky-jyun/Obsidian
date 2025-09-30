### 개념
const는 변수, 포인터, 함수, 객체 등을 상수로 사용할 수 있게 해준다.

const로 선언한 값은 메모리 내용을 변경할 수 없으며 변경하려는 시도가 있으면 컴파일 오류를 발생시킨다.

### 사용 목적
**1. 안전성**
- 값이 변경되는 것을 방지하여 버그 발생 가능성을 줄인다. (주요 목적)

**2. 가독성**
- 읽기 전용이라는 의도를 명확히 명시하여 다른 사람들이 이해하기 쉽게 한다.

**3. 최적화**
- 컴파일러는 값을 미리 계산하거나 레지스터에 저장하여 직접 값으로 대체하는 등 최적화를 진행할 수 있다.


---
### 상수 변수
**<mark style="background: #FFCC6C;">지역 변수</mark>**
const 변수는 반드시 선언과 동시에 초기화 해주어야 한다.
```cpp title:상수
const int MAX_NUM = 10;
const float PI = 3.14f;
```

**<mark style="background: #FFCC6C;">멤버 변수</mark>**
멤버 변수 또한 생성자에서 초기화 리스트를 사용해 초기화를 해주어야 한다.
```cpp
class Player{
private:
	const int age;
public:
	Player() : age(20); //초기화 리스트를 사용한 초기화
	
	/*Player(){
		// 구현부 내에서 초기화는 error
		age = 30; 
	}*/
};
```

---
### 상수 포인터
포인터는 `*`를 기준으로 `const`의 위치에 따라 의미가 달라진다.

const는 왼쪽을 수식한다는것을 기억하자.
단, const 왼쪽에 아무것도 없을 시 바로 오른쪽의 것을 수식하게 된다.

##### <mark style="background: #FFCC6C;">const *</mark>
`const int* p;` 처럼 `*`의 왼쪽에 `const`가 올 경우 `const`는 바로 오른쪽의 int 타입을 수식하게 된다.

`const int` 가 뜻하는것은 상수형 int란 뜻으로 `const int` 자체가 포인터가 가리키는 변수 타입이 된다.

그럼 `(const int)* p;`는 상수형 int 타입을 가리키는 포인터로 해석할 수 있다.

즉, 포인터가 가리키는 주소의 내용을 변경할 수 없다.

<span style="color:rgb(125, 125, 125)">(같은 이유로 `int const* p;` 도 동일하게 사용할 수 있다.)</span>

##### <mark style="background: #FFCC6C;">* const</mark>
`int* const p;` 처럼 `*`의 오른쪽에 `const`가 올 경우 `const`는 바로 왼쪽의 `*`포인터를 수식하게 된다.

`* const`가 뜻하는 것은 상수형 포인터란 뜻으로 포인터가 가리키는 주소 자체를 변경할 수 없다.

##### <mark style="background: #FFCC6C;">const * const</mark>
`const int* const p;` 처럼 `*`의 좌우에 `const`가 올 경우는 두가지 의미를 내포한다.
1. 상수형 타입을 내용으로 가진다.
2. 상수 표인터이다.

그래서 포인터가 가리키는 주소 & 내용 전부 변경할 수 없다.

---
### 상수 함수
#### 1. 상수 멤버 함수
상수 멤버 함수는 해당 함수 내에서는 this(객체)의 상태가 변할 수 없다는 약속이기도 하다.

하여 상수 멤버 함수는 구현부에서 멤버 변수 수정이 불가능해 멤버 변수 읽기용 함수에 잘 사용된다. <span style="color:rgb(125, 125, 125)">(구현부에서 지역 변수 생성 및 수정은 가능)</span>

함수 뒤에 const를 붙여 사용한다.
`반환타입 함수이름() const;`

```cpp title:상수멤버함수 hl:6
class Player{ 
private:
	int hp;
public:
	int getHP() const{
		this->hp = 3; // error
		int value = this->hp; // 지역 변수 선언 및 정의 가능
		return value;
	}
};
```

#### 2. 상수 멤버 함수 오버로딩
상수 멤버 함수는 const의 유무 차이로 오버로딩이 가능하다. 

그 이유는 멤버 함수마다 매개변수로 보이지 않는 this 변수를 가지는데 const의 유무에 따라 해당 변수 타입이 바뀌기 때문이다.

>**<mark style="background: #D2B3FFA6;">멤버 함수의 this 매개변수</mark>**
>멤버 함수는 눈에 보이지 않지만 매개변수에 객체 포인터 타입인 this 변수를 가진다.
>
>컴파일러는 `void print();`를 `void print(Rect* this);`로 보는 것이다.
>
>**<mark style="background: #D2B3FFA6;">상수 멤버 함수의 this 매개변수</mark>**
>상수 멤버 함수의 경우 눈에 보이지 않는 매개변수 this의 타입을 `const 객체클래스* this` 처럼 상수 객체 포인터로 바꿔버린다. 
>
>컴파일러는 `void print() const;`를 `void print(const Rect* this);`로 본다.

```cpp title:상수멤버함수오버로딩
class Rect{ 
pubic:
	void print() { cout << "normal" << endl; }
	void print() const { cout << "const" << endl; }
};

int main(){
	Rect rect;
	const Rect rectCon;
	
	rect.print(); // 출력: normal
	rectCon.print(); // 출력: const
}
```

##### <mark style="background: #FFCC6C;">객체와 상수 객체의 타입 변환</mark>
위의 코드 `상수멤버함수오버로딩`을 보면 객체는 멤버 함수를 호출하고, 상수 객체는 상수 멤버 함수를 호출하는 것을 볼 수 있다.

반대로 객체가 상수 멤버 함수를 호출하고, 상수 객체가 멤버 함수를 호출하게 되면 어떻게 될까?

일반 객체가 상수 멤버 함수를 호출하는것은 아무 문제가 되지 않지만 상수 객체가 일반 멤버 함수를 호출하는 것은 컴파일 오류를 발생시킨다.

```cpp title:상수멤버함수오버로딩오류 hl:9
class Rect{   
pubic:
	void print() { cout << "normal" << endl; }
};

int main(){
	const Rect rectCon;
	// 상수 객체가 일반 멤버 함수 호출
	rectCon.print(); // Error: 
}
```

그 이유는 바로 타입 변환에 있다.

함수마다 각각 숨겨진 this 타입의 매개변수가 있다는 것은 위에서 공부했을 것이다. 일반 객체로 함수를 호출할 경우, 인자로 일반 객체를 넘겨주게 되는데 일반 객체는 const 객체 타입에 호환이 된다. 

즉, `일반 객체 -> const 객체` 타입 변환 가능하다.
<span style="color:rgb(125, 125, 125)">(읽기 전용으로 잠시 변경하겠다는 의미_가능)</span>

반면 상수 객체로 함수를 호출할 경우, 인자로 상수 객체를 넘겨주게 되는데 상수 객체는 일반 객체 타입에 호환되지 않는다.

즉, `const 객체 -> 일반 객체` 타입 변환 불가능 하다.
<span style="color:rgb(125, 125, 125)">(읽기 전용을 쓰기 전용으로 변경하겠다는 의미_불가능)</span>

정리하자면 일반 객체는 const 객체로 타입 변환이 가능하고, 상수 객체는 일반 객체로 타입 변환이 불가능하여 위의 코드에서 컴파일 오류가 발생하는 것이다.

#### 3. 멤버 함수의 상수 매개 변수
함수의 매개 변수에서 복사 비용을 줄이기 위해 참조를 사용한다. 이때, 참조는 원본이 훼손될 위험이 있어  주로 const와 같이 사용된다.

**상수 객체 주의점**
- 상수 객체의 멤버 변수는 읽기가 가능하지만 수정은 불가능하다.
- 상수 객체의 멤버 함수는 상수 멤버 함수만 호출할 수 있다. <span style="color:rgb(125, 125, 125)">(상수 객체는 일반 멤버 함수를 호출할 수 없다._타입 변환 불가능)</span>
```cpp title:상수매개변수 hl:11,15
class Rect{ 
public:
	int length = 5;
	int getLength() const { return length; }
	void printLength() { cout << length << endl; }
};

void rectRef(const Rect& rect){
	// 멤버 변수 
	cout << rect.length << endl; // 읽기 가능
	rect.length = 10; // error: 값 수정 불가능
	
	// 멤버 함수
	int len = rect.getLength(); // 호출 가능
	rect.printLength(); // error: 호출 불가능
}
```

#### 4. 멤버 함수의 상수 리턴
멤버 함수에서 상수를 리턴할 때 const를 사용할 수 있다.
허나 일반 값을 상수로 리턴하는 것은 큰 의미가 없다.

##### <mark style="background: #FFCC6C;">상수 포인터 리턴</mark>
주로 객체 내부의 멤버 변수를 가리키는 읽기 전용 포인터를 외부에 제공하는데 사용된다.

`const T* 함수명() const;`

>**상수 멤버 함수만 상수를 리턴할 수 있나?**
>상수를 리턴한다고 해서 반드시 상수 멤버 함수일 필요는 없지만, 읽기 전용이라는 목적을 명시하기 위해 거의 함께 쓰인다.
>
>드물지만 함수가 상수를 리턴하기 전, 객체의 내부 상태를 변경해야 할 때 일반 멤버 함수가 상수를 리턴할 수 있다.
```cpp title:상수포인터리턴 hl:5,6
class Player{ 
private: 
	Weapon equippedWeapon; 
public: 
	const Weapon* getWeapon() const { 
		return &equippedWeapon; 
	} 
};
```

##### <mark style="background: #FFCC6C;">상수 참조 리턴</mark>
상수 포인터 반환 시 nullptr을 반환할 수도 있다.

허나 상수 참조 반환 시 nullptr을 반환할 수 없어 위험성을 줄일 수 있다. 

`const T& 함수명() const;`

```cpp title:상수참조리턴 hl:5-7
class Player {  
private: 
	string name; 
public: 
	const string& getName() const {
		// 복사 없이 'name'의 읽기 전용 별명을 리턴 
		return name; 
	} 
};
```

---
### define와 차이점
define은 전처리기 지시어로 코드에서 심볼을 찾아 텍스트를 대체(복사/붙여넣기)한다.

붙여넣기 시, define은 타입이 없어 타입과 관련된 오류를 잡아낼 수 없다. 

그러므로 상수는 define 대신 const를 사용하자.

define은 헤더 가드(ifndef ... endif) 조건부 컴파일에 사용하자.