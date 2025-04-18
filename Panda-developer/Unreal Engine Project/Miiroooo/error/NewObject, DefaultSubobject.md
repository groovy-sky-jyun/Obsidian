### Error

프로젝트를 진행하면서 아래 사진과 같은 오류를 접하게 되었다.

분명 빌드도 문제없이 되었고, 언리얼 프로젝트도 문제없이 실행되었다.

단, GamePlay() 도중에 플레이어가 아이템에 Overlap했을 때 자꾸 아래와 같은 Crash가 발생하였다.

![[Pasted image 20250306232117.png|EXCEPTION_ACCESS_VIOLATION]]

처음에는 오류가 발생한 코드에 문제가 있는것 같아서 코드들을 확인해 보았다.

오류가 발생한 코드는 아래와 같았다.

```cpp
// .h
TArray<AUsableItem*> OverlapItems;

// .cpp
void AddItem(AUsableItem* Item)
{
	OverlapItems.Add(); //error
}
```

---

#### 오류 해결 시도(1)

아무리 생각해도 TArray의 Add함수에서 오류가 났다는게 이상해서 검색을 해보니 EXCEPTION_ACCESS_VIOLATION 오류가 메모리 접근 문제일 수도 있다고 하였다. 뭔가 프로젝트 도중 꼬였나라는 생각에 생성자에 TArray 배열 초기화를 추가해주었다.

```cpp
// .h
TArray<AUsableItem*> OverlapItems;

// .cpp
UItemComponent()
{
	OverlapItems.Empty(); //추가
}
void AddItem(AUsableItem* Item)
{
	OverlapItems.Add(); //error
}
```

#### 오류 해결 시도(2)

UItemComponent 클래스는 기존에 UActorComponent 에서 작동했던 코드를 UObject로 변경한것이다.

코드는 같으므로 달라진건 상속받는 클래스의 차이 뿐이였다. 

그래서 기존의 저장된 데이터들과의 충돌이 발생한건가 싶어 Binaries, Intermediate, DerivedDataCache 파일들을 지우고 솔루션 파일을 재생성 하였다.

---

### 오류 해결

외부적인 문제가 아닌 코드문제인것 같았다. 그것을 확인하기 위해 아래와 같이 Add 함수를 주석처리하고 로그만 출력되도록 해보았다.

```cpp
void AddItem(AUsableItem* Item)
{
	UE_LOG(LogTemp, Warning, TEXT("ItemComponent::AddItem Success");
	// OverlapItems.Add(); //error
}
```

코드를 수정하니 놀랍게도 Crash가 발생하지 않았다. 즉, Add 부분이 문제라는게 확실해졌다.

그래서 매개변수가 null이 전달되는건가? 라는 생각이 들어 매개변수가 null이 아닌지 확인을 해보았다.

```cpp
void AddItem(AUsableItem* Item)
{
	if(Item)
    {
    	UE_LOG(LogTemp, Warning, TEXT("ItemComponent::AddItem Success");
    }
    else
    {
    	UE_LOG(LogTemp, Warning, TEXT("Item is null");
    }
	
	// OverlapItems.Add(); //error
}
```

그러나 Item은 알맞게 값이 잘 들어오고 있는것을 확인했다.

그럼 이제 확인해보지 않은곳은 AddItem을 호출하는 곳이다.

PlayerCharacter에서 ItemComponent->AddItem()을 호출하고 있기에 PlayerCharacter 클래스의 코드를 확인해보았다.

```cpp
APlayerCharacter::APlayerCharacter()
{
	...
    ItemComponent = NewObject<UItemComponent>(); //인스턴스 생성
}
APlayerCharacter::OverlapStart(...)
{
	ItemComponent->AddItem(OtherActor);
}
```

여기에서 의심스러운 부분은 생성자에서 ItemComponent의 인스턴스를 생성해주는 부분이였다.

기존의 ItemComponent는 UActorComponent를 상속받고 있었기에 Actor에 부착해주는 과정이 필요했다.

그래서 NewObject 대신 DefaultSubobject를 사용하여 APlayerCharacter에 부착해서 사용해주었다.

하지만 UObject로 변경하고 나서는 NewObject로 인스턴스로 생성해주었기에 이전과 다른부분이 이부분밖에 없었는 것이다.

확인해보니 <span style="color:rgb(255, 192, 0)">NewOject는 생성자에서는 사용할수가 없었다...</span>

하여 인스턴스 생성 부분을 BeginPlay() 함수로 옮겨주니 문제가 해결되었다.

> **NewObject**  
> - 런타임 중 동적으로 객체를 생성할 때 사용된다.  
> - UObject를 상속받는 클래스의 인스턴스를 생성한다.  
> - 생성자에서 호출해서는 안된다.  
>   
> **DefaultSubobject**  
> - CDO를 생성할 때 사용된다.  
> - 생성자에서 실행되며 클래스의 일부로 관리된다.  
> - 언리얼 에디터에 수정할 수 있으며, 런타임 중에 값이 변경되지 않는다.