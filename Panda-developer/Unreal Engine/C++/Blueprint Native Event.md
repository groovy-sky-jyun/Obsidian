#BlueprintNativeEvent 란 C++에서 구현한 함수를 Blueprint에서 오버라이딩 하여 사용할 수 있는 이벤트이다.

#### 사용 방법
UFUNCTION() 안에 "BlueprintNativeEvent" 태그를 추가하여 사용할 수 있는데 이때, 두가지 함수를 선언해주어야 한다.

1) Blueprint에서 오버라이딩 할 수 있는 함수명 = 'BP 서명'
2) C++에서 오버라이딩 할 수 있는 함수명 = 'C++ 서명'
	- 이름 끝에 `_Implementation` 접미사를 붙여주는것이 관행이다.
	- #VirtualFunction 이여야 한다. 소스 파일에서 이 함수를 구현해주어야 한다

```cpp title:MyActor.h
UFUNCTION(BlueprintNativeEvent)
void MyEvent(); 
virtual void MyEvent_Implementation();
```

```cpp title:MyActor.cpp
void MyActor::MyEvent_Implementation()
{
	UE_LOG(LogTemp, Warning, TEXT("MyEvent called in C++"));
}
```
