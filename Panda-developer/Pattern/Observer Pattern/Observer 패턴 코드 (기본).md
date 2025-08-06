### UI 체력 바 갱신 for Unreal Engine

```cpp title:HealthComponent.h
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

```cpp title:HealthBarWidget.h
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