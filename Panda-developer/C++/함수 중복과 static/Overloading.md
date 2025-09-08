### Overloading = 함수 중복 = 다형성

#### 적용 대상
- 전역 함수
- 멤버 함수
- 상속 관계 클래스

#### 조건
- 함수 이름이 동일해야 한다.
- 매개변수 <span style="color:rgb(255, 207, 61)">타입</span>이나 <span style="color:rgb(255, 207, 61)">개수</span> 중 하나라도 달라야 한다.
- 리턴 타입은 고려 대상이 아니다.
	- 리턴 타입이 다르더라도 매개변수 타입이나 개수 중 하나라도 다르지 않다면 컴파일 오류가 발생한다.

#### 실행 방법
중복 함수 호출 시, 컴파일러가 호출 매개변수와 일치하는 함수를 찾아 연결한다.
이때, 컴파일러는 리턴 타입을 고려하여 함수를 구분하진 않는다.

#### 예시
```cpp title:Overloading
int sum(int a, int b){ return a + b; }
double sum(double a, double b){ return a + b; }
int sum(int a, int b, int c){return a + b + c; }

int main(){
	cout << sum(2, 5);
	cout << sum(32.4, 56.2);
	cout << sum(2, 5, 7);
}
```
---

### 생성자 Overloading
매개변수를 통해 다양한 형태로 객체를 초기화 할 수 있다.
```cpp title:classOverloading
class Circle{
public:
	Circle();
	Circle(int r);
	//...
};

int main(){
	Circle dounut;
	Circle pizza(30);
}
```

### 소멸자 Overloading
소멸자는 매개변수를 가지지 않는다.
한 클래스에 한 개의 소멸자만 존재하므로 소멸자 함수 중복은 불가능하다.