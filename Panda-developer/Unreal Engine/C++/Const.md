### Const

const는 여러 상황에서 사용할 수 있으며 어떠한 상황에서는 사용하면 안된다.

아래는 여러 상황에 대한 예시이다.

**1) 포인터 자체를 const로 설정**

포인터를 const로 설정하면 다른 객체 주소의 재할당이 불가능하다.

그렇기에 한번 초기화되면 이후 다른 주소값을 가질 수 없다. (다른 객체를 가리킬 수 없다.)

이는 포인터가 가리키는 값이 고정돼 있다는것을 명시적으로 알려줌으로서 가독성을 높인다.

**주소값은 변경이 불가능**하지만 포인터가 가리키는 **객체의 데이터는 수정가능**하다.

```cpp
T* const Ptr = ...;	  //T는 수정 가능하나 포인터로의 재할당은 불가
Ptr->SomeFunction();  // Ptr이 가리키는 객체의 메서드를 호출하거나
Ptr->SomeValue = 5;   // Ptr이 가리키는 객체의 값을 수정할 수 있다.

T *Ptr //잘못된 사용방식
```

그럼 결과적으로 그냥 레퍼런스와 같은 동작을 하는것이 아닌가? 하는 의문점이 발생할 수 있다.

레퍼런스도 한번 초기화 되면 그 객체는 변경불가능하지만 객체의 데이터는 수정이 가능하기 때문이다. 

하지만 분명한 차이점이 존재한다.

아래의 코드를 살펴보자.

상수 포인터에 다른 객체를 넣는 것은 '포인터가 다른 객체를 가리키도록 하겠다'는 뜻이다.

레퍼런스에 다른 객체를 넣는 것은 '레퍼런스 객체 데이터값을 다른 객체의 데이터값으로 덮어씌우겠다'는 뜻의 차이가 있다.

```cpp
// 포인터 const 예시
void examplePointer()
{
    T object1, object2;
    T* const ptr = &object1;  // ptr은 object1을 가리킴, 다른 객체로 변경 불가

    ptr->SomeFunction();       // ptr이 가리키는 object1의 메서드를 호출
    ptr->SomeValue = 5;        // ptr이 가리키는 object1의 값을 수정

    // ptr = &object2;  // 컴파일 오류: 포인터는 다른 객체를 가리킬 수 없음
}

// 레퍼런스 예시
void exampleReference()
{
    T object1, object2;
    T& ref = object1;  // ref는 object1을 참조

    ref.SomeFunction();  // ref가 참조하는 object1의 메서드를 호출
    ref.SomeValue = 5;   // ref가 참조하는 object1의 값을 수정

    // ref = object2;  // object2의 데이터값을 ref가 참조하는 object1의 데이터값으로 덮어 씌움
}
```

**2) 레퍼런스를 const로 반환**

레퍼런스를 const로 반환하면 레퍼런스가 가지고 있는 **원본 데이터를 변경할 수 없도록** 지정해 주는 것이다.

```cpp
const TArray<FString>& GetSomeArray(); // 레퍼런스 반환
TArray<FString> SomeArray = GetSomeArray(); // 레퍼런스로 받아서 사용

SomeArray.Add(TEXT("New String"));  // 오류발생: 값 수정 불가능
```

**3) 포인터를 const로 반환**

포인터를 const로 반환하면 1) 와는 반대로 **포인터가 가리키는 객체의 데이터 값은 수정 불가능**하지만, **포인터 자체는 다른 객체(주소)를 가리킬 수 있다.** 즉, 포인터는 변수로서 재활용이 가능하다는 뜻이다.(포인터 재할당 가능)

```cpp
const TArray<FString>* GetSomeArray() {
    static TArray<FString> SomeArray;
    SomeArray.Add(TEXT("Hello"));
    SomeArray.Add(TEXT("World"));
    return &SomeArray;  // 배열의 주소를 반환하기 위해 & 사용
}

int main() {
    const TArray<FString>* Ptr = GetSomeArray();
    Ptr->Add(TEXT("New Item"));  // 오류: 배열 내용을 수정할 수 없음

    Ptr = nullptr;  // OK: Ptr 포인터는 다른 배열을 가리킬 수 있음
}
```

---

### 불필요한 const

**1) 레퍼런스에 const를 설정**

레퍼런스는 초기화된 값 이외의 객체를 가리킬 수 없기 때문에(변경이 불가능) const를 사용안하는 것과 하는것의 차이가 없다. 그렇기에 레퍼런스에 const를 설정하는것은 불필요한 일이다.

```cpp
T& const Ref = ...; // 잘못된 사용방식
```

**2) const를 반환**

객체의 복사 값을 const로 반환하면 반환된 값은 변경불가능하지만 원본 데이터는 변경될 수 있어 데이터 값이 맞지않는 문제가 발생할 수 있다. 그렇기에 이는 잘못된 사용법이다.

```cpp
const TArray<FString> GetSomeArray(); // 잘못된 사용방식
```

**3) const 포인터를 const로 반환**

const 포인터 자체가 주소값은 변경이 불가능하고, 데이터값은 변경이 가능하다. 그런데 이를 const로 반환하게 되면 데이터값마저 변경이 불가하게 되기 때문에 포인터의 목적성과 맞지 않고, 유연성을 제한한다.

```cpp
const TArray<FString>* const GetSomeArray(); // 잘못된 사용방식
```