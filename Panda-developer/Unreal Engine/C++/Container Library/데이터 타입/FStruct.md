구조체는 단순한 데이터 타입에 적합하다. 즉, 데이터를 저장/전송하는데 특화된 가벼운 객체이다.

구조체 이름은 F로 시작해서 언리얼 엔진이 메모리를 관리해주진 않는다.(포인터로 관리할 일이 없기 때문)

USTRUCT는 UPROPERTY를 지원하지만 UFUNCTION은 지원하지 않는다.

기본 구조체 틀은 아래와 같다.

주의할 점이 하나 있는데 오브젝트 포인터를 선언할 때는 반듯이 UPROPERTY()를 지정해줘야 자동으로 메모리 관리가 된다.

```cpp
USTRUCT(BlueprintType)
struct FStructName
{
	GENERATED_BODY()
    
    	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Test")
    	int32 id;
    
    	UPROPERTY() //오브젝트 포인터의 경우 필수
    	UObject* ObjectPointer;
};
```