컴포지션은 특정 객체에 다른 객체를 포함하여 사용하는 방식이다.

이를 통해 재사용성을 높일 수 있다.

상속은 부모관계를 나타내는 Is-A 관계를 가지지만, 컴포지션은 특정 객체가 다른 객체를 포함하는 Has-A 관계를 가진다.

오브젝트 입장에서 소유한 오브젝트를 Subobject, Subobject 입장에서 자신을 소유한 오브젝트를 Outer라고 한다.

컴포지션은 선언을 할 때 #include 대신 전방선언을 해주면 의존성을 줄일 수 있다.

전방선언은 생성부에서 아래 코드처럼 선언해준다.

```cpp
// 전방 선언
public:
	TObjectPtr<class UCube> Cube_1; // 언리얼5부터 변경됨 (선택사항)
    
    	class UCube* Cube_2; // 변경 전 전방 선언
```

---

### CreateDefaultSubobject

컴포넌트 객체를 생성할 때 사용한다.

정적으로 생성되며, 게임이 시작되거나 객체가 생성될 때 자동으로 초기화된다.

주로 Mesh나 조명 등을 액터에 추가하고자 할 때 사용된다.

생명주기는 액터가 파괴될 때 함께 파괴된다. 

```cpp
UStaticMeshComponent* MyMeshComponent = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("MyMeshComponent"));
MyMeshComponent->SetupAttachment(RootComponent);
```

> **컴포넌트 객체?**  
> 액터의 기능을 확장하기 위해 사용된다.  
> 주로 SetAttachment(RootComponent) 와 함께 사용되어 컴포넌트를 액터에 부착시켜 사용한다.

부모 생성자에서 CreateDefaultSubobject로 특정 컴포넌트 객체를 생성해 주었다면, 자식객체에서는 해당 컴포넌트 객체를 또 한번 생성할 필요없이 바로 이용할 수 있다.

```cpp
// 부모 생성자
AAnimal::AAnimal()
{
	// 컴포넌트 객체 생성
	Food = CreateDefaultSubobject<AFood>(TEXT("Food");
}


// 자식 생성자
APanda::APanda()
{
	// 컴포넌트 객체 생성 없이 바로 함수 사용
	Food->SetFoodType(EFoodType::Apple);
}
```

---

### NewObject

런타임에 객체를 동적으로 생성할 때 사용한다.

this를 설정해줌으로서 함수가 호출된 클래스를 부모로 설정하게 된다.

이렇게 생성된 객체는 가비지 컬렉션을 통해 메모리관리가 되기 때문에 부모 객체와는 독립적인 생명주기를 가진다.

```cpp
UMyCustomObject* NewObjectInstance = NewObject<UMyCustomObject>(this);
```

> **가비지 컬렉션?**  
> 객체가 더이상 참조되지 않으면 자동으로 메모리에서 해제하는 방식  
> 언리얼에서 제공하는 기능이다.