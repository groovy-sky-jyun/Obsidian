### 간략 설명
플레이어가 밟고 지나가면 흔들리다 무너지는 바닥 함정을 구현하였다.
무너지는 순간 밑에 Spike Trap에 닿여 Damage를 주는 함정이다.

바닥이 흔들리는 효과를 주기 위해서는 바닥을 여러개의 타일로 나눈 뒤, 타일을 서로 다르게 움직여야 한다. 이때, 타일의 움직임은 Rotation을 사용하여 원하는 효과를 낼 수 있다. (여기서는 4개의 타일로 나누도록 하겠다.)

Rotation을 계속해서 랜덤값으로 변경해주고, 타일이 서로 다른 Rotation을 가지도록 하면 바닥이 지진난 것 처럼 보이는 효과를 줄 수 있다.

---
### 구조
구조는 3개의 층으로 나누었다.

**_위층_**
Box Collision을 둬서 플레이어와 닿으면 바닥의 흔들림이 시작되도록 한다. 

**_중간 층_**
Floor Tile 4개를 붙여준다. <span style="color:rgb(125, 125, 125)">(Floor Tile을 1개씩만 했을 경우 플레이어와 Overlap된 Floor만 움직이게 된다. 그러면 땅이 무너지는 느낌이 덜 들기 때문에 Overlap됐을 때 4개 단위로 Floor가 움직이도록 하였다.)</span> 
Floor은 플레이어가 밟고 있는 바닥을 나타내며 아래 층의 Spike Trap을 가려주는 역할이기도 하다. 그래서 Top 시야에서 보았을 때 플레이어에게 Spike Trap이 보이지 않도록 해준다.

**_아래 층_**
Spike Trap을 둬서 플레이어가 닿으면 피가 닳도록 한다.

![[Group 64 1.png|600]]

--- 
### 동작 순서
1. 플레이어가 Overlap 되면 0.5초 뒤에 Wobbling 실행 (첫 번째 타이머 실행)
2. Wobbling 동안 DeltaTime을 이용해서 지속적으로 4개의 타일 Rotation 값을 랜덤 값으로 변경
	1. Wobbling이 실행되는 순간부터는 플레이어의 Overlap 유무와 상관 없이 1.5초간 실행 (두 번째 타이머 실행)
3. Wobbling이 끝나면 바닥은 무너지고 아래의 Spike Trap 나타남.

---

### 구현 방법
#### 1. Overlap Event
플레이어가 Overlap 되었다면 0.5초 타이머를 시작한다.
0.5초가 지나면 Wobbling이 무조건적으로 실행된다. (0.5초 전에 Overlap이 끝나도 Wobbling 실행)
```cpp hl:5,11
void ACollapsingTrap::PlayerOnFloorStart(UPrimitiveComponent* OverlappedComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
	if (OtherActor && OtherActor->IsA(APlayerCharacter::StaticClass()))
	{
		GetWorldTimerManager().SetTimer(DetectTimerHandle, this, &ACollapsingTrap::StartWobbling, 0.5f, false);
	}
}
```

<br>

#### 2. Start Wobbling
`bIsWobbling` 변수를 사용하여 Tick의 실행을 관리해준다. `bIsWobbling` 가 true일 경우 Tick의 함수가 계속 실행되고, false일 경우 실행되지 않는다.

Wobbling은 1.5초 동안 실행되어야 하므로 1.5초 뒤 `bIsWobbling`을 false로 바꿔주는 함수를 호출한다.
```cpp hl:5,14 
void ACollapsingTrap::Tick(float DeltaTime)
{ 
	Super::Tick(DeltaTime);

	if (bIsWobbling) 
	{
		WobblingTheTile();
	}
}

void ACollapsingTrap::StartWobbling()
{
	bIsWobbling = true;
	GetWorldTimerManager().SetTimer(WobbleTimerHandle, this, &ACollapsingTrap::EndWobbling, 1.5f, false);
}
```

Wobbling 효과를 줄 때, 4개의 Floor가 각기 다르게 회전해야 하므로 4개의 Floor Static Mesh를 배열에 담고, for문을 사용하여 배열 전체에 랜덤한 회전값을 적용시키도록 한다.

Floor의 Rotator(x, y, z) 을 #FRandRange 를 이용해 랜덤값으로 지정해 준다.


```cpp hl:5,7
void ACollapsingTrap::WobblingTheTile()
{  
	for (int i = 0; i < FloorArray.Num(); i++)
	{
		FRotator RandomRotation = FRotator(FMath::FRandRange(-20.0f, 20.0f), FMath::FRandRange(-20.0f, 20.f), FMath::FRandRange(-10.0f, 10.0f));
		
		FRotator NewRotation = FMath::Lerp(FloorArray[i]->GetComponentRotation(), InitRotationArray[i] + RandomRotation, 0.1f);
		
		FloorArray[i]->SetWorldRotation(NewRotation);
	}
}
```

> ##### <span style="color:rgb(188, 149, 218)">Rotator의 x, y, z값을 다 다르게 주는 이유</span>
> 위의 코드를 보면 x, y, z 각각에 Random 값을 다르게 주고 있다. 
> 랜덤값 하나를 x, y, z에 공통적으로 주는것과 x, y, z 각자 랜덤값을 주는것은 어떤차이가 있고, 왜 후자 방법을 택했는가?  
> 
> **_<span style="color:rgb(200, 184, 214)">랜덤값 하나를 공통으로 사용하는 경우</span>_**
> 랜덤값의 범위가 ±20 이라 가정하고, 정수만 생각해보자. (0 포함)
> 그러면 Floor 하나가 가질 수 있는 Rotator의 종류는 총 41가지 이다. 
> 
> **_<span style="color:rgb(200, 184, 214)">각 다른 랜덤값을 사용하는 경우</span>_**
> 랜덤값의 범위가 ±20 이라 가정하고, 정수만 생각해보자. (0 포함)
> Floor 하나가 가질 수 있는 Rotator의 종류는 41 x 41 x 41 = 68,921가지 이다.
>
> <span style="color:rgb(200, 184, 214)">**_결론_**</span> 
> 이렇게 나올 수 있는 경우의 수를 따졌을 때, 각 다른 랜덤값을 줬을때 훨씬 다양한 각도가 나올 수 있다. 
> 그러면 좀 더 불규칙스러운 움직임을 주므로써 플레이어에게 불안정한 긴장감을 줄 수 있다. 
> 그리고 바닥이 지진난것 처럼 흔들리는 것은 일정한 통일성을 갖지 않는게 훨씬 자연스러워 보일것이라 생각해 이 방법을 택하였다.
>
>
> ##### <span style="color:rgb(188, 149, 218)">z 값의 범위가 ±20이 아닌 ±10인 이유</span>
> z값은 수평에서의 비틀림 움직임을 나타낸다.
> 땅이 지진과 같은 진동을 받았을 때, 수평적 비틀림 보다는 (앞, 뒤, 좌, 우)의 움직임이 더 클것이라 생각하여 z값 범위를 x, y 값 범위보다 작게 주었다.

![[Wobbling.gif|]]

<br>

#### 3. End Wobbling
Wobbling은 1.5초 동안 실행된다. 1.5초가 지나면 Floor들은 바닥으로 무너져서 아래의 Spike Trap이 보여지게 된다. 그러기 위해 각 타일의 SimulatePhysics를 true로 줘서 중력을 통해 공중에서 떨어지는 것을 Floor가 무너지는 것처럼 보이게 사용한다.

그리고 Tick(Wobbling)이 더이상 실행되지 않도록 `bIsWobbling` 값을 `false`로 준다.
```cpp
void ACollapsingTrap::EndWobbling()
{
	bIsWobbling = false;

	SM_Floor_0->SetSimulatePhysics(true);
	SM_Floor_1->SetSimulatePhysics(true);
	SM_Floor_2->SetSimulatePhysics(true);
	SM_Floor_3->SetSimulatePhysics(true);

	TriggerBox->SetGenerateOverlapEvents(false);
}
```

![[Damage.gif]]