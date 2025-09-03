### 예시
몬스터가 크게 고블린, 오크 두 종류로 나뉜다고 가정해보자.
몬스터가 공통적으로 가지는 데이터는 다음과 같다.
`이름, 체력, 공격`

이를 프로토타입 패턴을 이용해 구현하려면 다음과 같은 데이터가 추가로 필요하다.
`ID, 몬스터 종류(클래스)`

이를 이용해 이름과 체력, 공격력이 조금씩 다른 고블린을 만들수도, 오크를 만들수도 있다. 큰 카테고리는 2개지만 이를 바탕으로 다양한 상태값을 가지는 객체들을 생성할 수 있다.

---

### 몬스터 데이터 에셋
몬스터의 기본 정보를 담는 클래스이다.
프로토타입 역할을 하며, 모든 몬스터가 가지는 공통된 데이터를 담고 있는 원형이다.
```cpp title:MonsterDataAsset hl:12
class ABaseMonster; 

class UMonsterDataAsset : public UDataAsset
{
	GENERATED_BODY()
	
public: 
	FName MonsterID;
	FText MonsterName;
	float BaseHealth;
	float BaseAttack;
	TSubclassOf<ABaseMonster> MonsterClass;
};
```
> ##### TSubclassOf<>
>데이터가 어떤 프로토타입 클래스를 기반으로 생성할지 정한다.

<br>

### 몬스터 스포너
실제 게임 월드에 배치되는 액터로, UDataAsset들을 관리하고 필요할 때마다 몬스터를 생성하는 공장 역할을 한다.
```cpp title:MonsterSpawner.h hl:25
class UMonsterDataAsset; 
class ABaseMonster; 

class AMonsterSpawner : public AActor
{
	GENERATED_BODY()
	
public:
	AMonsterSpawner();
	
	// 에디터에서 몬스터 데이터 에셋을 할당
	TArray<UMonsterDataAsset*> MonsterPrototypes;
	
protected:
	virtual void BeginPlay() override;
	
private:
	TMap<FName, UMonsterDataAsset*> PrototypeMap;
	
	// 몬스터 ID와 데이터를 연결시켜 놓는다.
	void InitializePrototypes();
	
public:
	// 몬스터 생성 및 초기화
	ABaseMonster* SpawnerMonster(FName MonsterID, const FVector& SpawnLocation);
};
```

```cpp title:MonsterSpawner.cpp hl:41-54
AMonsterSpawner::AMonsterSpawner() 
{
	PrimaryActorTick.bCanEverTick = false;
}

void AMonsterSpawner::BeginPlay()
{
	Super::BeginPlay();
	// 몬스터 ID와 데이터를 연결시켜 놓는다.
	InitializePrototypes(); 
	
	// 원본 객체 생성
	SpawnMonster(FName("Goblin"), GetActorLocation() + FVector(200.f, 0.f, 0.f));
	SpawnMonster(FName("Orc"), GetActorLocation() + FVector(-200.f, 0.f, 0.f));
}

void AMonsterSpawner::InitializePrototypes()
{
	for(UMonsterDataAsset* Data : MonsterPrototypes)
	{
		if(Data)
		{
			// MonsterID를 키로 하여 맵에 데이터를 추가한다.
			PrototypeMap.Add(Data->MonsterID, Data);
		}
	}
}

ABaseMonster* AMonsterSpawner::SpawnMonster(FName MonsterID, const FVector& SpawnLocation)
{
	// 1. map에서 몬스터의 원형 데이터(프로토타입)를 찾는다.
	UMonsterDatasset** PrototypeDataPtr = PrototypeMap.Find(MonsterID);
	if(!PrototypeDataPtr)
	{
		return nullptr;
	}
	
	UMonsterDataAsset* PrototypeDate = *PrototypeDataPtr;
	
	// 2. 새로운 액터를 생성한다. (clone)
	ABaseMonster* NewMonster = GetWorld()->SpawnActor<ABaseMonster>(
		PrototypeData->MonsterClass,
		SpawnLocation,
		FRotator::ZeroRotator,
		// 파라미터 생략
	); 
	
	// 데이터 초기화 (원형 데이터 복사)
	if(NewMonster)
	{
		NewMonster->Health = PrototypeData->BaseHealth;
		NewMonster->Attack = PrototypeData->BaseAttack;
		
		return NewMonster;
	}
	return nullptr;
}
```

> ###### 객체를 생성과 데이터 초기화를 따로하면 프로토타입 패턴이 아니지 않나요?
> 위 코드  `AMonsterSpawner::SpawnMonster()`를 보면 새로운 액터를 생성하고 난 뒤, 새 액터에 데이터를 초기화 하는 방식인 것을 볼 수 있다. 
>
> 프로토타입 패턴은 원형 객체의 데이터를 사용하는것인데 이러면 원형 데이터를 사용하는게 아니게 되지 않나? 라고 생각할 수 있다.
>
> 이는 언리얼 엔진의 특성때문이다. AActor 객체를 복사해야하는데 AActor는 new 연산만으로 완벽한 복제가 불가능하기 때문이다. 