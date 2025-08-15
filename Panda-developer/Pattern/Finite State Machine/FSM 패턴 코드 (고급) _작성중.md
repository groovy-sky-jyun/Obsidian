#### 문제 상황
캐릭터는 여러가지 상태를 가질 수 있다. 걷기, 뛰기, 점프 외에도 공격하기, 회피하기, 대쉬 등의 여러 상태를 가진다.

여러 종류의 상태들을 구현하고 전이 조건을 설정할 때, 상태가 많을수록 상태 패턴과 enum+switch문으로는 한계가 올 수 있다.

enum+switch문으로 구현할 경우 swith문이 거대해질 것이다.
상태 패턴으로 구현하는 경우에도 각 상태로 전환하는 부분이 복잡해질 것이다.

#### 해결 방안
이를 해결하기 위해 계층적 FSM을 사용할 수 있다.

우선, 캐릭터가 가질 수 있는 상태 종류를 분류해보자.
캐릭터가 가질 수 있는 상태는 크게 **이동, 공격, 피격** 등으로 분류할 수 있다. 이를 <span style="color:rgb(255, 192, 0)">최상위 상태</span>라고 하겠다.

각 최상위 상태는 여러개의 <span style="color:rgb(255, 192, 0)">하위 상태</span>를 가진다.
예를 들어 **이동**이라는 큰 카테고리 안에는 **걷기, 뛰기, 엎드리기** 등이 속한다. **공격**이라는 큰 카테고리 안에는 **찌르기, 주먹 휘두르기**가 속한다.

---

### 여러 상태 전이 구조
캐릭터가 다음과 같은 상태를 가진다고 했을 때, 상태 전이 코드 구조를 알아보자.
- MovementState : Idle, Run, Walk
- AttackState : Hit, Punch

#### 계층적 FSM 클래스 구조조
##### <mark style="background: #ADCCFFA6;">IState</mark>
- 모든 상태 클래스가 가지는 공통 인터페이스
- <span style="color:rgb(255, 192, 0)">OnEnter, OnUpdate, OnExit, HandleInput 정의</span>

##### <mark style="background: #ADCCFFA6;">SuperState</mark>
- <span style="color:rgb(255, 192, 0)">상위 상태 클래스가 공통으로 가지는 기능</span> 정의
- 즉, 공통적으로 사용되는 하위 상태 클래스 관리 기능 정의

##### <mark style="background: #ADCCFFA6;">MovementState, AttackState</mark>
- <span style="color:rgb(255, 192, 0)">상위 상태 클래스</span>
- MovementState는 이동과 관련된 <span style="color:rgb(255, 192, 0)">하위 상태 클래스를 묶어 관리</span>한다.
- AttackState는 공격과 관련된 하위 상태 클래스를 묶어 관리한다.

##### <mark style="background: #ADCCFFA6;">IdleState, RunState, JumpState, HitState, PunchState</mark>
- <span style="color:rgb(255, 192, 0)">하위 상태 클래스</span>

---

#### 1. 공통 인터페이스
```cpp title:IState
class IState{
public:
	virtual	void OnEnter(Character* owner) = 0;
	virtual void OnUpdate(Character* owner, float dt) = 0;
	virtual void OnExit(Character* owner) = 0;
	virtual void HandleInput(Character*owner, char input) = 0;
};
```
- 모든 상태 클래스가 가지는 공통 인터페이스이다.
- 모든 상태 클래스가 IState 클래스를 상속한다는 느낌이 아니고, <span style="color:rgb(255, 192, 0)">IState클래스를 인터페이스로 가짐으로써, 해당 클래스는 상태 클래스가 된다는 느낌</span>이다. (객체 지향 프로그래밍 원리)
- 즉, 모든 상태 클래스는 IState를 상속받아 상태로서의 역할을 수행할 수 있는 능력을 가지게 되는 것이다.

#### 2. 상위 클래스가 공통으로 관리하는 최상위 클래스스
```cpp title:SuperState hl:3
class SuperState : public IState {
protected:  
	std::unique_ptr<IState> mCurrentSubState;

public:
	void SetSubState(unique_ptr<IState> subState, Character* owner) {
		if(mCurrentSubState) {
			mCurrentSubState->OnExit(owner);
		}
		mCurrentSubState = std::move(subState);
		if(mCurrentSubState){
			mCurrentSubState->OnEnter(owner);
		}
	}

	void OnUpdate(Character* owner, float dt) override {
		// 하위 상태와 상관 없는 공통 로직
		if(owner->isDead()){
			owner->ChangeState(std::make_unique<DeadState>());
			return;
		}

		// 하위 상태의 OnUpdate 호출
		if(mCurrentSubState){
			mCurrentSubState->OnUpdate(owner, dt);
		}
	}

	void HandleInput(Character* owner, char input) override {
		// 어떤 상태에 있던 a 가 입력되면 공통으로 적용되는 전이 조건
		if(input == 'a'){
			owner->ChangeState(std::make_unique<AttackState>());
			return;
		}

		// 그 외의 입력
		if(mCurrentSubState){
			mCurrentSubState->HandleInput(owner, input);
		}
	}
};
```
- `SuperState`는 하위 상태 클래스들을 관리하기 위해 상위 클래스들이 공통으로 가져야 하는 관리 기능을 제공한다.
- `mCurrentSubState`는 현재 활성화된 하위 상태를 나타낸다.
- `SetSubState()`는 하위 클래스 전환을 담당한다.
- `OnUpdate()`와 `HabdleInput()`은 하위 상태와 무관한 공통 로직을 먼저 처리하고, 그 외의 로직은 하위 상태에 위임한다.
	- 예) 어떤 상태에 있던 a 키를 누르면 상위 클래스는 AttackState가 된다.
	- SuperState에 정의하지 않으면 상위/하위 클래스마다 해당 조건을 작성해 줘야 해서 코드 중복이 발생한다.


#### 3. 상위 상태 클래스 구현
```cpp title:MovementState
#include "SuperState.h"
#include "IdleState.h"
#include "WalkState.h"
#include "RunningState.h"

class MovementState : public SuperState{
public:
	void OnEnter(Character* owner) override {
		//이동 상태에 진입 시, 초기 하위 상태는 Idle로 설정
		//IdleState의 OnEnter 호출
		SetSubState(std::make_unique<IdleState>());
	}
	void OnUpdate(Character* owner, float dt) override{
		//...
	}
	void OnExit(Character* owner) override {
		//...
	}
	void HandleInput(Character* owner, char input) override {
		if(mCurrentSubState){
			mCurrentSubState->HandleInput(owner, input);
		}
	}
};
```



```cpp title:Movement하위클래스
#include "IState.h"

class Character;
class WalkingState;
class RunningState;

class IdleState : public IState{
public:
	void OnEnter(Character* owner) override{
		//Idle 애니메이션 시작
		stamina = 100.0f;
	}
	void OnUpdate(Character* owner, float dt) override{
		//...
	}
	void OnExit(Character* owner){
		//...
	}
	void HandleInput(Character*owner, char input) {
		if(input == 'w'){
			// 하위 상태를 Walking 으로 변경
			// MovementState의 SetSubState를 호출해야 한다.
			// owner->GetSuperState()->SetSubState(...)
		}
		else if(input == 'r'){
			// 하위 상태를 Running으로 변경
		}
	}
}

#include "IState.h"

class Character;
class RunningState;

class WalkingState : public IState{
public:
	void OnEnter(Character* owner) override{
		//걷기 애니메이션 시작
	}
	void OnUpdate(Character* owner, float dt) override{
		// location += walkeSpeed * dt;
		stamina -= 10.f * dt;
	}
	void OnExit(Character* owner){
		//...
	}
	void HandleInput(Character*owner, char input) {
		if(input == 'r'){
			// 하위 상태를 Running 으로 변경
			// owner->GetSuperState()->SetSubState(...)
		}
	}
};

#include "IState.h"

class Character;

class RunningState : public IState{
private:
	float stamina = 100.0f;
public:
	void OnEnter(Character* owner) override{
		//뛰기 애니메이션 시작
		stamina = 100.0f;
	}
	void OnUpdate(Character* owner, float dt) override{
		if(stamina <= 0){
			//스태미나가 0이 되면 Idle상태로 강제 전환
			//owner->GetSuperState()->SetSubState(...)
			return;
		}
		// location += runSpeed * dt;
		stamina -= 10.f * dt;
	}
	void OnExit(Character* owner){
		//...
	}
	void HandleInput(Character*owner, char input) {
		//...
	}
}
```

```cpp title:하위클래스
#include "IState.h"

class Character;
class MovementState;

class AttackState : public IState {
private:
	int attackComboCount = 0;
public:
	void OnEnter(Character* owner) override{
		//공격 애니메이션 시작, 콤보 초기화
		attackComboCount = 0;
	}
	void OnUpdate(Character* owner, float dt) override{
		//콤보 타이머, 데미지 판정 등 공격 관련 로직
		//애니메이션 끝나면 MovementState로 전환
		if(/*애니메이션 종료 시 */ == true){
			owner->ChangeState(std::make_unique<MovementState>());
		}
	}
	void OnExit(Character* owner) override {
		//공격 상태가 끝나면 실행될 로직
	}
	void HandleInput(Character* owner, char input) override{
		//공격 중 추가 입력 처리 (콤보 공격)
	}
}
```

#### 3. Character 클래스 수정
Character 클래스는 현재 상태에 따라 switch문으로 분기할 필요가 없다.
포인터를 통해 현재 상태 객체에게 모든걸 위임하도록 한다.
```cpp title:Character
#include "StateMachine.h"

class Character{
private:
	StateMachine mStateMachine;
	int mHP = 100;
	
public:
	Character()
	: mStateMachine(this){
		mStateMachine.SetState(std::make_unique<MovementState>());
	}

	void Update(float dt){
		if(mStateMachine){
			mStateMachine.Update(dt);
		}
	}

	// 입력 처리 예시 
	void HandleInput(char input){
		if(mStateMachine){
			mStateMachine.HandleInput(input);
		}
	}
};
```
- Character와 키입력을 분리하기 위해 HandleInput을 각 상태 클래스에서 구현하도록 하였다.
- Character는 키입력에 따라 어떤 상태가 되는지 알 필요가 없다.

#### StateMachine 분리
```cpp title:StateMachine
#include "IState.h"

class Character; //전방 선언

class StateMachine{
private:
	std::unique_ptr<IState> mCurrentState;
	Character* mOwner;

public:
	StateMachine(Character* owner)
	: mOwner(owner){}

	void SetState(std::unique_ptr<IState> mewState){
		if(mCurrentState){
			mCurrentState->OnExit(mOwner);
		}
		mCurrentState = std::move(newState);
		if(mCurrentState){
			mCurrentState->OnEnter(mOwner);
		}
	}

	void Update(float dt){
		if(mCurrentState){
			mCurrentState->OnUpdate(mOwner, dt);
		}
	}

	void HandleInput(char input){
		if(mCurrentState){
			mCurrentState->HandleInput(mOwner, input);
		}
	}
};

```

#### 4. 실행
```cpp title:main
int main(){
	Character player;
	player.Update(0.16f);

	player.HandleInput('w'); 
	player.Update(0.16f);
	player.Update(0.16f);

	player.HandleInput('r');
	player.Update(0.16f);
	player.Update(0.16f);

	return 0;

}
```

---

#### 장점
- 상태를 객체로 분리하여 낮은 결합도와 높은 응집도를 가진다.
- 새로운 상태를 추가하거나 기존 상태 로직을 변경 시, Character클래스를 건드릴 필요 없이 해당 상태 클래스만 수정하면 된다.

#### 주의사항
>##### 순환 참조 문제
>위의 코드를 구현할 때 Character 클래스에서 IState를 include 하고, 각각의 상태 클래스 또한 Charaxter 클래스를 include 하게 되면 순환참조 문제가 발생할 수 있다.
>
>##### 해결 방법
>이를 해결 하기 위한 방법은 전방 선언을 사용하는 것이다.
>
>IState 클래스의 멤버 함수들은 Character의 포인터만 매개변수로 받고 있다.
IState는 Character의 크기나 구체적인 멤버에 대해 알 필요가 없으므로 `#include "Charcter.h"` 대신 `class Character;` 라는 전방선언을 사용해준다. 

<br>

 > ##### #전방선언
> 헤더파일에서 특정 클래스의 포인터나 참조만 사용하면, 그 클래스의 헤더 파일을 include 할 필요 없이 전방선언으로 충분하다.
> 
> 하지만 그 클래스의 객체를 직접 생성하거나, 멤버에 접근, 크기를 알아야 할때는 반드시 헤더 파일을 include 해서 전체 정의를 알려주어야 한다.

---

#### 한계점
분명 잘 짜여진 코드이지만 한계가 명확히 존재한다.
플레이어는 한번에 하나의 상태만 가질 수 있다는 것이다.

본래 게임에서 플레이어는 서있는 상태에서 공격을 할 수도 있다.
점프를 하면서 공중 공격을 할 수도 있다.

이를 구현할 수 있으려면 어떻게 해야할까?

#### 해결 방법
계층적 FSM을 사용하면 된다.
자세한 설명은 다음 페이지에 작성되어 있다.
[[FSM 패턴 코드 (고급)]]