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