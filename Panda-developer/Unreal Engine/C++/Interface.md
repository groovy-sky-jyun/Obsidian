
인터페이스는 함수만을 가지는 클래스이다.

인터페이스를상속하는 오브젝트는 인터페이스에 선언된 함수들을 모두 구현해야하는 규칙을 가지고 있다.

특정 기능에 대해 오브젝트별로 다르게 구현되어야 할 때 용이하게 사용할 수 있다.

사용방법은 아래 예시를 통해 설명하겠다.

블루프린트 네이티브 이벤트와 인터페이스를 이용하여 체력이 0이 되었을 때 플레이어와 몬스터가 다르게 동작하는 것을 구현해보겠다.

#### 사용 방법

- HealthComponent는 Health와 관련된 함수들을 정의하는 Component이다.
- HealthInterface는 Health와 관련된 함수들 중 오버라이딩이 필요한 함수들을 모아둔 Interface이다.
- 플레이어와 몬스터는 각각 HealthComponent와 HealthInterface를 포함한다.

각각의 Actor가 HealthComponent를 포함하는 이유는 각 Actor마다 Health가 다르게 관리되기 때문이다.(Actor는 하나의 Health를 공유하지 않고, 각자의 Health를 가진다.)

![[Pasted image 20250306232417.png|클래스 구조]]



아래의 사진은 클래스들을 한눈에 보기 좋게 정리한 것이다.

![[Pasted image 20250306232425.png|인터페이스 구현 예시]]



**인터페이스**

인터페이스에서는 블루프린트 네이티브 이벤트를 사용해서 순수가상함수 OnDeath_Implementation() 을 선언하고 있다.

인터페이스를 포함하고 있는 PlayerCharacter와 Monster클래스는 각각 OnDeath_Implementation() 함수를 오버라이딩하고 있는 것을 볼 수 있다.

> **인터페이스 가상함수**  
> 함수 이름 끝에 접미사 **_Implementation** 을 붙인다.

**컴포넌트**

HealthComponent에서는 Health가 0이하가 되었을 때, Interface를 통해 구현된 함수를 호출하도록 하고 있다.

그러면 HealthComponent의 소유자에 따라 실행되는 함수가 달라지게된다.

- 플레이어가 Health가 0이 되면 플레이어가 소유하고 있는 HealthComponent의 LoseHealth() 함수 안에서 소유자인 플레이어가 오버라이딩한 OnDeath_Implementation() 함수가 실행된다.
- 몬스터의 Health가 0이 되면 몬스터가 소유하고 있는 HealthComponent의 LoseHealth() 함수 안에서 소유자인 몬스터가 오버라이딩한 OnDeath_Implementation() 함수가 실행된다.

> **컴포넌트에서 Interface를 통해 구현된 함수 호출**  
> **IMyInterface::Execute_MyEvent(GetOwner());**