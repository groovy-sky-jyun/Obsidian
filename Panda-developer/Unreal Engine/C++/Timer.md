### Timer Manager
#타이머 를 통해 일정 시간 딜레이 이후 동작을 수행하도록 하거나, 일정 시간 동안 특정 동작을 수행하도록 설정할 수 있다.

타이머는 Global Timer Manager에서 관리한다.
Global Timer Manager은 Game Instance 오브젝트와 World에 존재한다.

TimerManager는 타이머의 속도, 경과 시간, 남은 시간 등의 정보 획득을 위한 함수를 제공한다.

타이머를 설정하는데 사용되는 주 함수는 `SetTimer`와 `SetTimerForNextTick` 이있다. 

--- 
### 타이머 설정 및 해제
#TimerHandle 을 통해 다양한 작업을 할 수 있다. 아래는 다양한 작업 중 몇가지 예시이다.
![[Pasted image 20250421190007.png]]
- 카운트다운 일시정지(재개)
- 남은 시간 확인 및 변경
- 타이머 취소
```cpp
#include "TimerManager.h"

void MyActor::BeginPlay()
{
	Super::BeginPlay();
	
	FTimerHandle TimerHandle;
	// MyFunction 을 초당 1회, 지금부터 2초간 호출한다.
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