### 2. Observer Pattern for Unreal Engine

---
#### GameEventSubsystem
```cpp title:GameEventSubsystem.h hl:3,13
#include "Subsystems/GameInstanceSubsystem.h"

DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOnEnemyKilledSignature, AActor*, KilledEnemy);
 
UCLASS()
class UGameEventSubsystem : public UGameInstanceSubsystem
{
	GENERATED_BODY()

public:
	//다른 시스템들이 이 델리게이트에 구독한다.
	UPROPERTY()
	FOnEnemyKilledSignature OnEnemyKilled;
}
```

#### EnemyCharacter
```cpp title:EnemyCharacter.cpp hl:12
#include "GameEventSubsystem.h"
 
void AEnemyCharacter::Die()
{
	//기존 사망 이펙트, 애니메이션 ...

	//전역 이벤트 알림 : 적 처치 
	if(UWorld* World = GetWorld())
	{
		if(UGameEventSubsystem* Events = World->GameInstance()->GetSubsystem<UGameEventSubsystem>())
		{
			Events->OnEnemyKilled.Broadcast(this);
		}
	}

	Destroy();
}
```

#### AchievementManager
```cpp title:AchievementManager.h
#include "Subsystems/GameInstanceSubsystem.h"
 
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOnAchievementUnlocked, FName, AchievementID);

UCLASS()
class UAchievementManager : public UGameInstanceSubsystem
{
	GENERATED_BODY()

public:
	//옵저버들이 이 델리게이트에 구독한다.
	UPROPERTY()
	FOnAchievementUnlocked OnAchievementUnlocked;

	//초기화 시점에 이벤트 바인딩
	virtual void Initialize(FSubsystemCollectionBase& Collection) override;

	//적 처치 이벤트 핸들러
	UFUNCTION()
	void HandleEnemyKilled(AActor* KilledEnemy);

	//업적 해제 호출
	void UnlockAchievement(FName AchievementID);

private:
	bool bFirstKill { false };
	int32 KillCount { 0 };
	const int32 RequiredKillForMaster { 10 };
}
```

```cpp title:AchievementManager.cpp
#include "AchievementManager.h"
#include "GameEventSybsystem.h"

void UAchievementManager::Initialize(FSubsystemCollectionBase& Collection)
{
	Super::Initialize(Collection);

	// 적 처치 이벤트에 구독
	if(UGameEventSubsystem* Events = GetGameInstance()->GetSubsystem<UGameEventSybsystem>())
	{
		Events->OnEnemyKilled.AddDynamic(this, &UAchievementManager::HandleEnemyKilled);
	}
}

void UAchievementManager::HandleEnemyKilled(AActor* KilledEnemy)
{
	// 첫 처치 업적
	if(!bFirstKill)
	{
		UnlockAchievement("FirstKill");
		bFirstKill = true;
	}

	// 누적 처치 업적
	KillCount++;
	if(KillCount >= RequiredKillForMaster)
	{
		UnlockAchievement("KillMaster");
	}
}

void UAchievementManager::UnlockAchievement(FName AchievementID)
{
	//업적 해제 델리게이트 발행
	OnAchievementUnlocked.Broadcast(AchievementID);
}

private:
	bool bFirstKill { false };
	int32 KillCount { 0 };
	const int32 RequiredKillForMaster { 10 };
}
```


##### Observer (UI Widget)
```cpp title:AchievementWidget.h
#include "AchievementManager.h"

UCLASS()
class UAchievementWidget : public UUserWidget
{
	GENERATED_BODY()
	
public:
	UPROPERTY()
	UAchievementManager* AchievementManager;

	virtual void NativeConstruct() override
	{
		Super::NativeConstruct();

		if(AchievementManager)
		{
			AchievementManager->OnAchievementUnlocked.AddDynamic(this, &UAchievementWidget::HandleAchievementUnlocked);
		}
	}

protected:
	UFUNCTION()
	void HandleAchievementUnlocked(FName AchievementID)
	{
		UE_LOG(LogTemp, Log, TEXT("UI: 업적 해제: %s"), *AchievementID.ToString());
	}
};
```

---

##### Observer (Save Component)
```cpp title:AchievementSaveComponent.h
#include "Components/ActorComponent.h"
#include "AchievementManager.h"

UCLASS()
class UAchievementSaveComponent : public UActorComponent
{
	GENERATED_BODY()

public:
	UPROPERTY()
	UAchievementManager* AchievementManager;

	virtual void BeginPlay() override
	{
		Super::BeginPlay();
		
		if(AchievementManager)
		{
			AchievementManager->OnAchievementUnlocked.AddDynamic(this, &UAchievementSaveComponent::HandleAchievementUnlocked);
		}
	}

protected:
	UFUNCTION()
	void HandleAchievementUnlocked(FName AchievementID)
	{
		UE_LOG(LogTemp, Log, TEXT("Save: 업적 저장 - %s"), *AchievementID.ToString());
	}
};
```
