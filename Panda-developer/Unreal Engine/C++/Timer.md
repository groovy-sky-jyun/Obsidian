### #FTimerManager
타이머 를 통해 일정 시간 딜레이 이후 동작을 수행하도록 하거나, 일정 시간 동안 특정 동작을 수행하도록 설정할 수 있다.

타이머는 Global Timer Manager에서 관리한다.
Global Timer Manager은 Game Instance 오브젝트와 World에 존재한다.

TimerManager는 타이머의 속도, 경과 시간, 남은 시간 등의 정보 획득을 위한 함수를 제공한다.

타이머를 설정하는데 사용되는 주 함수는 `SetTimer`와 `SetTimerForNextTick` 이있다. 

#FTimerHandle 을 통해 다양한 작업을 할 수 있다. 아래는 다양한 작업 중 몇가지 예시이다.
- 카운트다운 일시정지(재개)
- 남은 시간 확인 및 변경
- 타이머 취소

--- 
### #SetTimer 타이머 설정 및 해제

![[Pasted image 20250421190007.png]]
매개변수를 통해서 여러 타이머 설정을 할 수 있다.
- InOutHandle : FTimerHandle을 입력받는다.
- InObj : 실행하는 클래스(객체)
- InTimerMethos : CallBack함수로 지정된 타이머 시간이 끝나면 자동으로 호출된다.
- InRate : 몇초 간격으로 실행할지를 정한다.
- InbLoop : 타이머를 반복으로 실행할지 정한다. (flase = 한 번 호출)
- InFirstDelay : 가장 처음 실행 시간은 몇 초 뒤에 InTimerMethos를 실행할 지 정한다.
	- (반복일 경우) 처음은 InFirstDelay 시간이 지난 뒤 함수를 호출하고, 그 뒤로는 InRate 간격으로 함수를 호출한다.
	- (생략할 경우) InRate 타임을 Default 값으로 가진다.
	- 예) InRate가 5초이고, InFirstDelay가 2초라면 첫 함수 호출은 2초 뒤에 실행된다. 그 뒤 5초 간격으로 반복 호출된다.

```cpp
#include "TimerManager.h"

void MyActor::BeginPlay()
{
	Super::BeginPlay();
	
	FTimerHandle TimerHandle;
	// MyFunction 2초 뒤 첫 호출, 이후는 1초 간격 호출
	GetWorldTimerManager().SetTimer(TimerHandle, this, &MyActorClass::MyFunction, 1.0f, true, 2.0f);
}
void MyActor::MyFunction()
{
	// 함수를 충분히 호출했으면 타이머 해제
	if(--RepeatingCallsRemaining <= 0)
	{
		GetWorldTimerManager().ClearTimer(TimerHandle);
		// 이후 TimerHandle은 다른 타이머에서 재사용 가능하다.
	}
}
```

>__타이머 해제__
>`SetTimer` 를 0이하의 속도로 호출하는 것은 ClearTimer를 호출하는 것과 같다.

---

### SetTimerForNextTick
타이머를 시간 간격이 아닌 다음 프레임에 실행되도록 설정할 수 있다.
이 함수는 Timer Handle을 채우지 않는다.

---

### 타이머 일시정지 및 재개
TimerHandle의 `PauseTimer` 를 이용하여 타이머를 일시정지한다.
그럼 타이머는 시간 카운트가 멈추지만, 경과 시간과 남은 시간은 그대로 유지된다.
`UPauseTimer` 는 일시정지된 타이머를 재개한다.
- `IsTimerActive` 함수를 이용해 타이머가 현재 활성화 되어 있는지 확인할 수 있다.
- `GetWorldTimerManager().IsTimerActive(TimerHandle);`
---

### 사용 예시
플레이어가 파워 업을 했을 경우 5초간 속도가 빨라졌다가 다시 원래 속도로 돌아가는 기능을 TimerManager를 사용해서 구현해보자.
```cpp hl:11 
void PowerUp()
{ 
	GetCharacterMovement()->MaxWalkSpeed = 500.f;
	UWorld* World = GetWorld();
	World->GetTimerManager().SetTimer(PowerHandle, this, &Player::EndPowerUp, 5.0f, false);
}

void EndPowerUp()
{
	GetCharacterMovement()->MaxWalkSpeed = 300.f;
	World->GetTimerManager().ClearTimer(PowerHandle);
}
```

> ##### #ClearTimer 를 하는 이유?
> Clear Timer를 해주지 않으면 타이머 실행은 끝났지만 Timer Handle은 계속해서 해당 타이머를 가리키고 있다.  이는 원치 않는 동작을 일으킬 수 있기에 안전성을 위해서 Clear Timer 사용을 권장한다.
> 
> Clear Timer를 해주면 Timer Handle은 아무것도 가리키지 않게 되어 재사용이 가능하다. 물론, Clear Timer를 해주지 않아도 다른 타이머로 덮어 씌우는게 가능하지만 원치 않는 동작을 막기 위해 Clear Timer를 사용한다.