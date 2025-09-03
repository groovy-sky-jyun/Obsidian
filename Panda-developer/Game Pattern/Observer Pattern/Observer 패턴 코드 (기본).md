### UI 체력 바 갱신 for Unreal Engine
#### Subject : HealthComponent
```cpp title:HealthComponent.h hl:1,18 
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOnHealthChanged, float, NewHealthRatio);  

UCLASS()
class UHealthComponent : public UActorComponent
{
	GENERATED_BODY()

public:
	UPROPETY()
	FOnHealthChanged OnHealthChanged;

	UFUNCTION()
	void TakeDamage(float DamageAmount)
	{
		CurrentHealth = FMath::Clamp(CurrentHealth - DamageAmount, 0.f, MaxHealth);

		//체력 브로드캐스트
		OnHealthChanged.Broadcast(CurrentHealth / MaxHealth);
	}

private:
	float MaxHealth = 100.f;
	float CurrentHealth = 100.f;
}
```

- <span style="color:rgb(255, 192, 0)">델리게이트를 선언</span>하고, 상태 변화(`TakeDamage`)가 일어날 때 <span style="color:rgb(255, 192, 0)">Broadcast로 알림을 보낸다.</span>

>**델리게이트**
> - 내부적으로 등록된 모든 옵저버(리스너) 함수 포인터 목록을 보관
> - Broadcast 호출 시 포인터 목록을 순회하며 각 콜백 호출

---

#### Observer : HealthBarWidget
```cpp title:HealthBarWidget.h hl:18
UCLASS()
class UHealthBarWidget : public UUserWidget
{
	GENERATED_BODY()

public:
	UPROPERTY()
	UProgressBar* HealthBar;

	UPROPERTY()
	UHealthComponent* HealthComp;

	virtual void NativeConstruct() override
	{
		Super::NativeConstruct();
		if(HealthComp)
		{
			HealthComp->OnHealthChanged.AddDynamic(this, &UHealthBarWidget::HandleHealthChanged);
		}
	}

protected:
	UFUNCTION()
	void HandleHealthChanged(float NewRatio)
	{
		HealthBar->SetPercent(NewRatio); 
	}
} 
```

- `OnHealthChanged` <span style="color:rgb(255, 192, 0)">델리게이트에 메서드 바인딩 (구독)</span>
- Subject가 Broadcast 할 때마다 `HandleHealthChanged` 가 호츨되어 UI를 갱신한다.

<br>

>##### 구독 취소
몬스터가 죽기 전 구독을 취소하는 코드를 추가해보자.
> ```cpp
> virtual void NativeDestruct() override
> {
>	// 파괴 직전 해제
>  if (HealthComp)
>   {
>	    HealthComp->OnHealthChanged.RemoveDynamic(this, &UHealthBarWidget::HandleHealthChanged);
>	}
>	        Super::NativeDestruct();
>}
>```
>
>언리얼엔 가비지 컬렉션이 있다. 객체가 죽으면(Destroy) 속해있는 HealthBarWidget 또한 Destroy 되므로, 바인딩한 델리게이트는 자동으로 끊겨 불필요한 콜백 호출이 없다.
>
>허나, UI 위젯같이 화면에 나타났다가 사라지기만 할 뿐, 객체 자체가 살아있는 상황이라면 어떨까? UI가 화면에서 사라졌을 때는 알림을 받을 필요가 없으므로 의도적으로 구독을 취소해주어야 한다.