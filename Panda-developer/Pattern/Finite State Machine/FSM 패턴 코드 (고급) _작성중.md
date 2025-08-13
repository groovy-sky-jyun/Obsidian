## 계층적 FSM
이전까지는 캐릭터의 이동과 관련하여 구현해보았지만 좀 더 확장시켜보자.

우선, 캐릭터가 가질 수 있는 상태 종류를 분류해보자.
캐릭터가 가질 수 있는 상태는 크게 **이동, 공격, 피격** 등으로 분류할 수 있다. 이를 <span style="color:rgb(255, 192, 0)">최상위 상태</span>라고 하겠다.

각 최상위 상태는 여러개의 하위 상태를 가진다.
예를 들어 **이동**이라는 큰 카테고리 안에는 **걷기, 뛰기, 엎드리기** 등이 속한다. 

#### 클래스 구조
**IState**
- 모든 상태 클래스가 가지는 공통 인터페이스
- OnEnter, OnUpdate, OnExit, HandleInput 정의

**SuperState**
- 상위 상태 클래스가 공통으로 가지는 기능 정의
- 즉, 공통적으로 사용되는 하위 상태 클래스 관리 기능 정의

**MovementState, AttackState**
- 상위 상태 클래스
- MovementState는 이동과 관련된 하위 상태 클래스를 묶어 관리한다.
- AttackState는 공격과 관련된 하위 상태 클래스를 묶어 관리한다.

**IdleState, RunState, JumpState, HitState, PunchState**
- 하위 상태 클래스

---

#### 1. 최상위 상태 클래스 
```cpp title:IState
class IState{
public:
	virtual	void OnEnter(Character* owner) = 0;
	virtual void OnUpdate(Character* owner, float dt) = 0;
	virtual void OnExit(Character* owner) = 0;
	virtual void HandleInput(Character*owner, char input) = 0;
};
```

```cpp title:SuperState
class SuperState : public IState {
protected: 
	std::unique_ptr<IState> mCurrentSubState;

public:
	void SetSubState(unique_ptr<IState> subState) {
		if(mCurrentSubState) {
			mCurrentSubState->OnExit(nullptr);
		}
		mCurrentSubState = std::move(subState);
		if(mCurrentSubState){
			mCurrentSubState->OnEnter(nullptr);
		}
	}

	void OnUpdate(Character* owner, float dt) override {
		// 하위 상태 공통 로직
		if(mCurrentSubState){
			mCurrentSubState->OnUpdate(owner, dt);
		}
	}

	void HandleInput(Character* owner, char input) override {
		if(mCurrentSubState){
			mCurrentSubState->HandleInput(owner, input);
		}
	}
};
```


#### 2. 최상위 상태 클래스 구현
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
		// 공격 키가 눌리면 AttackState로 변경
		if(input == 'a'){
			owner->ChangeState(std::make_unique<AttackState>());
			return;
		}

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
class Character{
private:
	std::unique_ptr<IState> mCurrentState;

public:
	Character(){
		//시작 상태를 Idle로 설정
		mCurrentState = std::make_unique<IdleState>();
		mCurrentState->OnEnter(this);
	}

	void Update(float dt){
		if(mCurrentState){
			mCurrentState->OnUpdate(this, dt);
		}
	}

	// 상태 변경
	void ChangeState(std::unique_ptr<IState> newState){
		if(mCurrentState){
			mCurrentState->OnExit(this);
		}
		
		mCurrentState = std::move(newState);
		mCurrentState->OnEnter(this);
	}

	// 입력 처리 예시 
	void HandleInput(char input){
		/* 입력에 따른 상태 전환은 여기서 해도 되고,
		* 각 State 클래스의 Update 안에서 처리해도 된다.*/
		if(mCurrentState){
			mCurrentState->HandleInput(this, input);
		}
	}
};
```
- Character와 키입력을 분리하기 위해 HandleInput을 각 상태 클래스에서 구현하도록 하였다.
- Character는 키입력에 따라 어떤 상태가 되는지 알 필요가 없다.


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