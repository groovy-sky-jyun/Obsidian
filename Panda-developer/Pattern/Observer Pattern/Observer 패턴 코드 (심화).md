### 적 처치 및 업적 달성 시스템 for UnrealEngine
<span style="color:rgb(125, 125, 125)">(게임 내에서 일어나는 이벤트들은 `GameEventSubsystem`클래스에서 관리된다고 가정한다.)</span>

### 1. 적 처치
플레이어가 적을 죽였을 때, 다음과 같은 이벤트들이 발생할 수 있으며, 관련된 클래스들은 다음과 같다.
- 적 처치 업적 달성 -> AchievementManager
- Enemy Die Effect -> SoundManager
- Enemy Die Sound -> VFXManager

##### 실행 흐름
1) 적이 처치되면 세개의 이벤트가 발생한다.
2) 알림은 GameEventSubsystem에서 관리한다.
3) 그러므로 GameEventSubsystem에 `적 처치` Delegate를 추가한다.
4) AchievementManager, SoundManager, VFXManager은 `적 처치` 를 구독한다.

![[OnEnemyKilled 구독 관계도.png|'적 처치' 실행 흐름]]

<br>

### 2. 업적 달성 
여기에서 `AchievementManager`를 좀 더 살펴보자.
업적이 달성되면 다음과 같은 이벤트들이 발생할 수 있으며, 관련된 클래스들은 다음과 같다.
- 업적 달성 알림 띄우기 -> AchievementWidget
- 업적 목록 Update (달성된 업적은 달성으로 변경) -> AchievementSaveComponent

##### 실행 흐름
1. 업적이 달성되면 두개의 이벤트가 발생한다.
2. 해당 알림은 업적 로직에 속하므로 AchievementManager에 `업적 해제` Delegate를 추가한다.
3. AchievementWidget, AchievementSaveComponent는 `업적 해제` 를 구독한다.

![[OnAchievementUnlocked 구독 관계도.png|'업적 달성' 실행 흐름]]

<br>

### 3. 전체 흐름
![[실행 순서.png|적 처치 및 업적 달성 흐름]]

---
### 코드 구현

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
- 적이 죽으면 GameEventSubsystem의 `OnEnemyKilled` 델리게이트를 호출한다.

<br>

#### GameEventSubsystem
```cpp title:GameEventSubsystem.h hl:3,13
#include "Subsystems/GameInstanceSubsystem.h"

DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOnEnemyKilledSignature, AActor*, KilledEnemy);
 
UCLASS()
class UGameEventSubsystem : public UGameInstanceSubsystem
{
	GENERATED_BODY()

public:
	//적 처치 델리게이트트
	UPROPERTY()
	FOnEnemyKilledSignature OnEnemyKilled;

	//이 외의 여러 델리게이트
	//...
}
```
- 적 처치 델리게이트를 추가해준다.
- 해당 클래스는 시스템과 관련된 여러 델리게이트를 관리한다.

<br>

<span style="color:rgb(125, 125, 125)">(SoundManager, VFXManager은 BeginPlay()에서 GameEventSubsystem의 OnEnemyKilled 델리게이트를 구독해준다. 이하 생략... )</span>

<br>

#### AchievementManager
```cpp title:AchievementManager.h hl:3,13
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

```cpp title:AchievementManager.cpp hl:11,35
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
```
- GameEventSubsystem의 `OnEnemyKilled` 델리게이트를 구독한다.
- 어디에선가 `OnEnemyKilled`가 호출되어 알림을 보내면 바인딩된 `HandleEnemyKilled` 메서드가 실행된다.
- 업적 달성 델리게이트(`OnAchievementUnlocked`)를 추가한다.
- 조건이 만족되어 업적이 달성되면 `OnAchievementUnlocked` 델리게이트를 호출한다.

<br>

##### AchievementWidget, AchievementSaveComponent
```cpp title:AchievementWidget.h hl:20
#include "AchievementManager.h"
 
UCLASS()
class UAchievementWidget : public UUserWidget
{
	GENERATED_BODY()
	
public:
	UPROPERTY()
	UAchievementManager* AchievementManager;

	// AchievementSaveComponent는 BeginePlay에 작성
	// 내용은 동일
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
- AchievementManager의 `OnAchievementUnlocked`를 구독한다.
- `OnAchievementUnlocked`가 어딘가에서 호출되어 알림을 보내면 바인딩된 `HandleAchievementUnlocked` 메서드가 실행된다. 