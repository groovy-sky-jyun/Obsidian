## TMap

TMap<KeyType, ValueType> 형식으로 데이터를 저장하면 key, value가 한 쌍으로 이루어져 저장되는 컨테이너이다.

즉, TMap에서의 element는 키-값을 뜻한다.

TMap의 특징 중 하나는 중복값저장이 되지 않는다. 그렇기에 Key는 고유한 값을 가지게 된다.

만약 기존의 Key값과 일치하는 데이터를 추가하면 Value가 새로운 값으로 대체된다. 

다른 특징으로는 해시 컨테이너이기에 값을 찾는데 유용하다. 대신 순서를 보장하지 않는다.

마지막에 새 element를 삽입하였다고해서 그 element의 위치가 항상 끝에오는게 아니란 뜻이다.

중간 데이터를 삭제하더라도 순번이 앞으로 당겨지지 않기에 TArray와 달리 빈 공간이 발생할 수 있다.

---

#### 구조체를 Key로 설정

구조체를 키로 명시하고자 할때 GetTypeHash 함수와 operator(==)을 override 해주어야 한다.

그렇지않으면 컴파일 에러가 발생한다.

방법은 간단하다.

Struct 안에 아래 operator와 GetTypeHash 함수를 설정해주면 된다.

아래의 코드는 ItemStructure가 3개의 변수값을 가졌을 때 ItemStructure를 키로 설정하는 코드이다.

```cpp
struct FItemStructure
{
	UPROPERTY()
    	int32 Index;
    
    	UPROPERTY()
    	FName Name;
    
    	UPROPERTY()
    	int32 Damage;
}

// operator 설정 (Index와 Name을 묶어서 키로 설정)
bool operator==(const FItemStructure& Other)
{
	return Index == Other.Index && Name == Other.Name;
}

// 해시 함수 (Index와 Name을 묶어서 키로 설정)
friend FORCEINLINE uint32 GetTypeHash(const FItemStructure& Item)
{
	uint32 Hash = 0;
    	Hash = HashCombine(Hash, GetTypeHash(Item.Index)); 
    	Hash = HashCombine(Hash, GetTypeHash(Item.Name));  
    	return Hash;
        
        // Index 하나만 Key로 설정하고자 할 때
        // return GetTypeHash(Item.Index);
}
```

> **GetTypeHash**  
> TMap은 GetTypeHash 함수로 해시값을 만든다.  
> 위에서는 ItemStructure의 Index와 Name을 Hash값 만드는데 사용해주었다.  
> 이 방법은 다양한 방법이 있으니 다른 방법들도 궁금하다면 검색해보길 바란다.   
>   
> **operator==**  
> TMap에서는 비교를 할 때 해시값을 이용해 위치를 먼저 찾는다.  
> 그런 뒤 operator==를 통해 실제 키 값을 비교하게된다.   
> 위에서는 Index와 Name값만 비교하는것으로 설정하였다.