값이 바뀌지 않는것을 명시하는 키워드이다.
객체를 상수화 한다는 뜻으로 객체의 데이터 변경을 허용하지 않는다.

const 객체는 const 멤버함수 호출만 가능하다.

const 함수 내에서는 const 함수만 사용이 가능하다.

### const 포인터
```cpp
int main(void) {
	int a = 1, b = 2;

	const int* ptr1 = &a; //값 수정 불가, 주소 변경 가능
	int* const ptr2 = &a; //값 수정 가능, 주소 변경 불가능
	const int* const ptr3 = &a; //값 수정 불가능, 주소 변경 불가능
}
```

### const 멤버 함수
```cpp
class Car {
public:
	string name;
	void print() const {
		//this->name = "hi"; 멤버변수 변경 불가능
	}
};
```

### const 참조
```cpp
class Car {
public:
	string name;
	void print(const string& s) {
		// s += "!"; ❌ 변경 불가
		cout << s << endl;
	}
};
```