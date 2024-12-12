구조체
```cpp
USTRUCT()
struct API FColoredTexture
{
	GENERATED_USTRUCT_BODY()
public:
	
}
```

구조체의 특징
- 가비지 컬렉션의 관리를 받지 않는다.
- BP 에서 사용하는 함수를 만들 수 없다.(함수 직렬화 불가능)
- UObject보다 가볍다.

enum
```cpp
UENUM()
enum Statuc
{
	Stopped UMETA(DisplayName = "Stopped"),
	Moving UMETA(DisplayName) = "Moving"),
	Attacking UMETA(DisplayName) = "Attacking")
};
```

``` cpp
UPROPERTY(EditAnywhere, BlueprintReadWrite)
TEnumAsByte<Status> status;
```
TEnumAsByte 는 열거형의 정수화된 값(uint8 0~255)을 가진다. 이는 메모리 효율을 높여준다. 이 값을 GetDisplayNameText를 통해 DisplayName을 가져올 수 있다.
``` cpp
MyEnum = EMyEnum::Vlue2 //내부적으로는 정수값을 가짐
uint8 RawValue = MyEnum.GetValue(); //정수값 반환
```