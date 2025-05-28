### 간략 설명
두개의 Spike Trap이 서로 마주보고 있고, 서로 거리가 가까워졌다가 멀어졌다가를 반복하는 함정을 구현하였다. 설명하기에 앞서 편리하게 서로 마주보는 Spike Trap을 왼쪽 벽과 오른쪽 벽이라 칭하겠다. 

플레이어가 사이를 지나가려다 양쪽 중 하나의 Spike Trap에 닿으면 Damage를 준다.
이때, 왼쪽과 오른쪽 벽에 동시에 Overlap이 되더라도 한번의 Damage만 받게 된다.
Overlap이 끝나고 다시 Overlap이 되는 조건에서는 무조건 Damage를 받는다.

서로 거리가 일정한 속도로 가까워지다가 일정 거리가 되면 쾅! 하고 닫아서 플레이어가 두 벽 사이에 있을 경우 껴서 못 빠져나가도록 한다.

---
### 구조
구조는 2개의 서로 마주보는 Static Mesh과 각각의 Static Mesh에 Collision을 부착하는 형태로 구성하였다.

두 개의 Static Mesh은 계속해서 이동한다. 이때, Collision도 같이 이동해야 하므로 각각의 Static Mesh에 Box Collision을 부착 시켜주었다. 그렇지 않으면 Collision은 그대로 있고, Static Mesh만 계속 움직이게 된다.

![[Group 65.png|600]]

--- 
### 동작 순서
1. 두 개의 벽 사이 거리가 MinDistance 값 이하가 될 때까지 서로 가까워진다.
	1. 일정한 속도로 이동한다.
	2. 두 개의 벽 사이 거리가 (MinDisntace + α) 이하가 되면 속도를 빠르게 바꿔 쾅! 하고 플레이어를 가둬버린다.
	3. 두 벽이 MinDistance 값 이하가 되면 0.5초 정지했다가 2번을 실행한다.
2. 두 개의 벽 사이 거리가 MinDistance 값 이하가 되면, MaxDistance 값 이상이 될 때까지 서로 멀어진다.
	1. 일정한 속도로 이동한다.
	2. 두 벽이 MaxDistance 값 이상이 되면 1.2~2초 사이의 랜덤 시간만큼 정지했다가 1번을 실행한다.
3. 1번과 2번을 반복한다.
4. 동작 과정과 별개로 벽의 Spike Trap에 플레이어가 닿으면 Damage를 입는다.
	1. 플레이어가 두 개의 벽에 동시에 닿았을 경우에는 Damage를 한번만 입는다.
	2. 몬스터는 Damage를 받지 않는다.

---

### 구현 방법
#### 1. 초기값 설정
이 함정에서 중요한 부분은 두 벽 사이의 거리라고 생각했다. 어디까지 좁혀졌다가 어디까지 멀어질지, 어느 구간에서 가속도를 줄지 구상이 필요했다. 하여 아래의 그림과 같이 디자인 하였다.

![[Group 68.png]]

**MaxDistance**
우선 `MaxDistance`는 기존 두 벽 사이의 거리 값을 사용하였다.
이때, Relative Location이 x와 z의 값 중 하나라도 0이 아닌 경우가 있을 수 있기에 StartLocation.Y를 사용하지 않고, 벡터의 크기를 사용하도록 했다.
하여 `MaxDistance = FMath::Abs(StartLocation.Y - (-StartLocation.Y))` 대신 `MaxDistance = (StartLocation - (-StartLocation)).Size()` 를 사용하였다.

**Weight**
`Destination`을 설정하기에 앞서, `Destination`의 위치를 설정하기 위한 비율을 구하기 위해 `Weight`를 작성했다.
두 벽 사이의 가운데 점으로 부터 (전체 거리 x 1 / 3) 만큼 떨어진 위치를 `Destination`으로 지정하고자 하였다. (전체 거리 x 2 / 5 ) 등의 숫자를 적용해 보았으나 1 / 3을 곱했을 때가 가장 적절하였다.

**Destination**
`Weight`값을 이용하여 `StartLocation`으로 부터 `Weight`만큼 떨어진 위치를 `Destination`으로 설정해 주었다.

**MinDistance**
왼쪽 벽과 오른쪽 벽의 Destination 사이의 거리를 `MinDistance`로 주었다.
`MaxDistance`와 마찬가지로 x와 z 값 중 하나라도 0이 아닌 경우를 대비하여 벡터의 크기를 사용하였다.

**Acceleration**
가속도를 주는 구간을 설정해 주어야 했는데 이는 두 벽 사이의 거리가 `MinDistance`의 1.8배 남았을 때의 위치로 적용해 주었다.

```cpp hl:14,22
void AWallTrap::BeginPlay()
{
	Super::BeginPlay();
 
	R_StartLocation = R_SpikeWall->GetRelativeLocation();
	L_StartLocation = L_SpikeWall->GetRelativeLocation();
		
	R_CurrentLocation = R_StartLocation;
	L_CurrentLocation = L_StartLocation;

	R_DestinationLocation = R_StartLocation;
	L_DestinationLocation = L_StartLocation; 
	
	MaxDistance = (R_StartLocation - L_StartLocation).Size() - 1.f; 
	
	//두 wall 사이의 1/3 위치에  DestinationLocation 설정 (y값만)
	float DistanceWeight = FMath::Abs(MaxDistance * (1.0f / 3.0f));
	float Direction = (L_StartLocation.Y < R_StartLocation.Y) ? 1.0f : -1.0f;
	R_DestinationLocation.Y = R_StartLocation.Y - (DistanceWeight * Direction);
	L_DestinationLocation.Y = L_StartLocation.Y + (DistanceWeight * Direction);
	
	MinDistance = (R_DestinationLocation - L_DestinationLocation).Size() + 1.f;

	Accelerator = MinDistance * 1.8f;

	bIsClosing = true;
	bIsStop = false;
	LerpAlpha = 0.0f;
}
```

>##### <span style="color:rgb(188, 149, 218)">코드 상에서 MaxDistance와 MinDistance에 ±α 를 해주는 이유</span>
> ±α를 하지 않은 상태에서 함정을 실행해보면 두 벽이 가까워지다가 `if (CurrentDistance <= MinDistance)` 조건을 만족하면 멀어져야 하는데, 해당조건이 영원히 만족되지 않는 결과가 발생한다. 
> 
> 이는 후의 코드에서 나오겠지만 Lerp로 CurrentDistance를 Destination까지 움직이는 코드에서 Lerp의 동작방식에 의해 alpha값을 1.0f로 주지 않는 이상 CurrentDistance는 Destination에 도달하지 못한다.
>
>CurrentDistance에서 MaxDistance로 도달하는것 또한 같은 이유로 불가능하기에 ±α로 조절해준 것이다.
>
>이는 다음 3.Moving The Walls 에서 더 자세히 다루도록 하겠다.

<br>

#### 2. Tick
두 벽이 가까워 졌다가 멀어지는 움직임은 계속 반복해야 하므로 Tick에서 구현해준다.
`bIsClosing`로 가까워지는 상태/멀어지는 상태를 표현한다.
`bIsStop`로 Start Location이나 Destination Location에 도달했을 때, 잠시 멈추어 있는 상태를 표현한다.
```cpp
void AWallTrap::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	if (!bIsStop)
	{
		if (bIsClosing) //가까워짐
		{
			ClosingTheWalls(DeltaTime);
		}
		else //멀어짐
		{
			OpeningTheWalls(DeltaTime);
		}
	}
}
```

<br>

#### 3. Moving The Walls
Lerp를 이용하여 현재 위치에서 목적지(Start Location/Destination Location) 까지 일정 속도로 움직이는 동작을 구현하였다.
```cpp
void AWallTrap::MovingTheWalls(float Alpha, FVector R_Destination, FVector L_Destination)
{
	FVector LeftLocation = FMath::Lerp(L_CurrentLocation, L_Destination, Alpha);
	FVector RightLocation = FMath::Lerp(R_CurrentLocation, R_Destination, Alpha);

	L_SpikeWall->SetRelativeLocation(LeftLocation);
	R_SpikeWall->SetRelativeLocation(RightLocation);

	R_CurrentLocation = R_SpikeWall->GetRelativeLocation();
	L_CurrentLocation = L_SpikeWall->GetRelativeLocation();

	CurrentDistance = FMath::Abs(R_CurrentLocation.Y - L_CurrentLocation.Y);
}
```
 
> ##### <span style="color:rgb(188, 149, 218)">#Lerp에서 Current가 Destination이 될 수 없는 이유</span>
> 위의 1. 초기값 설정에서 말했다시피 Lerp에서 Alpha =1.0이 아니면 목적지에 영영 도달할 수 없는 문제가 발생한다. 그 이유를 살펴보자.
>
> 아래는 언리얼에서 Lerp를 구현한 코드이다.
> A를 Current, B를 Destination이라고 생각해보자.
> Current와 Destination 사이의 거리를 Alpha 비율만큼 Current에서 더해줌으로써 Destination에 가까워지는 동작을 한다.
> 
>  ```cpp title:UnrealMathUtility.h
> T Lerp( const T& A, const T& B, const U& Alpha )
> {
>	return (T)(A + Alpha * (B-A));
>}
> ```
>
>이 코드를 바탕으로 Current = 0 부터 시작하여 Destination = 100 까지 Alpha = 0.4 의 비율로 이동하는 그래프를 보자. <span style="color:rgb(125, 125, 125)">(데이터 값을 적게 사용해서 그래프 곡선이 매끄럽지 않다.)</span>
>
>![[Pasted image 20250515220043.png|400]]
>
>그래프의 끝부분분을 확인해보면 Current Location이 100, 남은 거리가 0처럼 보인다. 하지만 표를 확인해보자.
>
>|Step|Current|남은 거리|
>|---|---|---|
>|0|0|100|
>|1|40|60|
>|2|64|36|
>|3|78.4|21.6|
>|4|87.04|12.96|
>|5|92.224|7.776|
>|6|95.3344|4.6656|
>|7|97.20064|2.79936|
>|8|98.32038|1.679616|
>|9|98.99223|1.00777|
>|10|99.39534|0.604662|
> 이처럼 Alpha값이 1.0이 되지 않는 이상 Current는 100에 가까워지기만 하고, 남은 거리또한 0에 가까워지기만 한다. 그래서 Lerp는 Destination에 도달할 수 없는 것이다.
> 
> 그래서 이를 보완하기 위해 MaxDistance와 MinDistance에 ±α를 해준 것이다.

<br>

> ##### <span style="color:rgb(188, 149, 218)">왜 Lerp를 사용하였는가?</span>
> 언리얼에서 보간을 다루는 함수는 다양하게 있다. 위의 목적지에 도달하지 못하는 Lerp의 대안을 찾다가 VInterpTo를 알게 되었다. 
> 
> **#VInterpTo**
> VInterpTo는 언리얼에서 아래와 같이 구현되어 있다.
> ```cpp title:UnrealMathUtility.h
> CORE_API FVector FMath::VInterpTo( const FVector& Current, const FVector& Target, float DeltaTime, float InterpSpeed )
> {
>	// If no interp speed, jump to target value
>	if( InterpSpeed <= 0.f )
>	{
>		return Target;
>	}
>
>	// Distance to reach
>	const FVector Dist = Target - Current;
>
>	// If distance is too small, just set the desired location
>	if( Dist.SizeSquared() < UE_KINDA_SMALL_NUMBER )
>	{
>		return Target;
>	}
>
>	// Delta Move, Clamp so we do not over shoot.
>	const FVector	DeltaMove = Dist * FMath::Clamp<float>(DeltaTime * InterpSpeed, 0.f, 1.f);
>
>	return Current + DeltaMove;
> }
> ```
> 코드를 분석해보자. return의 조건은 3가지를 가진다.
> 1) InterpSpeed가 0 이하가 되면 바로 Target을 반환한다.
> 2) Target와 Current사이의 거리가 일정 사이즈 미만이 되면 Target을 반환한다. (Lerp와 다른 점)
> 3) `Current + DeltaMove` 를 반환한다.
> 	- DeltaMove : (Target와 Current 사이의 거리) x (DeltaTime x InterpSpeed)
> 	- (DeltaTime x InterpSpeed)의 값은 0~1 이다.
> 	- DeltaTime을 0.01, InterpSpeed를 5.0 이라 가정하면 0.05가 된다.
> 
> 그럼 일정거리 이하가 되면 바로 Destination을 반환해주니까 VInterpTo를 사용하면 되겠네? 라고 생각했지만 놓친 부분이 있었다.
> 
> **<span style="color:rgb(188, 149, 218)">VInterpTo는 일정한 속도를 끝까지 유지할 수 없다.</span>**
> DeltaMove를 구할 때, Target와 Current 사이의 거리에 A 
> 









<br>

#### 4. Closing The Walls
벽은 Current Location에서 Destination Location 방향으로 이동한다.
서로 가까워질 때는 3가지 조건에 따라 각자 다른 이벤트가 발생한다.

1) Current Distance <= MinDistance
	- 0.5초동안 멈췄다가 Opening The Walls를 실행한다.
2) Current Distance <= Accelerator
	- 가속도를 줘서 플레이어가 예상하지 못한 동작을 준다.
	- 쾅! 하고 두 벽이 닫히는 듯한 연출을 준다.
3) 그 외의 상황
	- 일정한 속도로 두 벽의 사이가 가까워진다.

```cpp hl:8,14,20
void AWallTrap::ClosingTheWalls(float DeltaTime)
{ 
	if (CurrentDistance <= MinDistance) //멀어짐으로 변경
	{
		bIsStop = true; 

		FTimerHandle OpenTimerHandle;
		GetWorldTimerManager().SetTimer(OpenTimerHandle, this, &AWallTrap::OpeningTimer, 0.5f, false);
	}
	else if (CurrentDistance <= Accelerator) //가속
	{
		float AccelerationAlpha = DeltaTime * 0.2f + 0.2;
		LerpAlpha = FMath::Clamp(AccelerationAlpha, 0.0f, 1.0f);
		MovingTheWalls(LerpAlpha, R_DestinationLocation, L_DestinationLocation);
	}
	else //일정속도로 닫힘
	{
		float SpeedAlpha = DeltaTime * 0.2f + 0.006; 
		LerpAlpha = FMath::Clamp(SpeedAlpha, 0.0f, 1.0f); 
		MovingTheWalls(LerpAlpha, R_DestinationLocation, L_DestinationLocation);
	}
}
void AWallTrap::OpeningTimer()
{
	bIsClosing = false;
	bIsStop = false;
}
```

<br>

#### 5. Opening The Walls
벽은 Current Location에서 Start Location 방향으로 이동한다.
서로 멀어질 때는 2가지 조건에 따라 각자 다른 이벤트가 발생한다.

1) CurrentDistance >= MaxDistance
	- 1.2 ~ 2초 랜덤 값만큼 멈췄다가 Closing The Walls를 실행한다.
2. 그 외의 상황
    - 일정한 속도로 두 벽의 사이가 멀어진다.

```cpp hl:9,15
void AWallTrap::OpeningTheWalls(float DeltaTime)
{   
	if (CurrentDistance >= MaxDistance) //가까워짐으로 변경
	{
		bIsStop = true;

		float RandomTimer = FMath::FRandRange(1.2f, 2.0f);
		FTimerHandle CloseTimerHandle;
		GetWorldTimerManager().SetTimer(CloseTimerHandle, this, &AWallTrap::ClosingTimer, RandomTimer, false);
	}
	else //일정속도로 열림
	{
		float SpeedAlpha = DeltaTime * 0.2f + 0.008;
		LerpAlpha = FMath::Clamp(SpeedAlpha, 0.0f, 1.0f);
		MovingTheWalls(LerpAlpha, R_StartLocation, L_StartLocation);
	}
}

void AWallTrap::ClosingTimer()
{
	bIsClosing = true;
	bIsStop = false;
}
```

<br>

#### 6. Overlap
플레이어와 Overlap이 되었을 경우 Damage를 주면된다.
이때, 두 벽과 동시에 Overlap되었을 경우 Damage는 한번만 주어야 한다.
하여 각각의 Collision Box Overlap에 Damage함수를 바인딩 해주었다.

```cpp
AWallTrap::AWallTrap()
{
	R_CollisionBox->OnComponentBeginOverlap.AddDynamic(this, &AWallTrap::BeginOverlapTrap);
	L_CollisionBox->OnComponentBeginOverlap.AddDynamic(this, &AWallTrap::BeginOverlapTrap);
}

void AWallTrap::BeginOverlapTrap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
	APlayerCharacter* Player = Cast<APlayerCharacter>(OtherActor);
	if (Player)
	{
		HitDamage(OtherActor, Damage);
	}
}
```

**<span style="color:rgb(255, 100, 100)">문제</span>**
두 벽에 플레이어가 동시에 Overlap되면 2번의 Damage가 들어가는 문제가 발생했다.

**<span style="color:rgb(128, 202, 255)">해결 방법</span>**
데미지를 입었는 플레이어를 관리하는 배열을 추가해 주었다.
배열에 추가된 플레이어는 배열에서 없어질 때까지 추가 데미지를 받을 수 없다.

그럼 동시에 두 벽에 Overlap 되었더라도 미세하게 먼저 Overlap된 Collision Box가 먼저 배열에 플레이어를 추가할 것이다.
후에 Overlap된 Collision Box는 이미 배열에 Overlap된 플레이어가 담겨있기 때문에 조건 불충족으로 플레이어에게 데미지를 줄 수 없게 된다.

Overlap이 끝나면 배열에서 해당 플레이어를 삭제해야 하므로 EndOverlap 바인딩을 추가해 주었다.

```cpp
AWallTrap::AWallTrap()
{
	R_CollisionBox->OnComponentBeginOverlap.AddDynamic(this, &AWallTrap::BeginOverlapTrap);
	R_CollisionBox->OnComponentEndOverlap.AddDynamic(this, &AWallTrap::EndOverlap);
	
	L_CollisionBox->OnComponentBeginOverlap.AddDynamic(this, &AWallTrap::BeginOverlapTrap);
	L_CollisionBox->OnComponentEndOverlap.AddDynamic(this, &AWallTrap::EndOverlap);
}

void AWallTrap::BeginOverlapTrap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
	APlayerCharacter* Player = Cast<APlayerCharacter>(OtherActor);
	if (Player && !DamagedPlayers.Contains(Player))
	{
		HitDamage(OtherActor, Damage);
		DamagedPlayers.Add(Player);
	}
}

void AWallTrap::EndOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex)
{
	APlayerCharacter* Player = Cast<APlayerCharacter>(OtherActor);
	if (Player && DamagedPlayers.Contains(Player))
	{
		DamagedPlayers.Remove(Player);
	}
}
```
