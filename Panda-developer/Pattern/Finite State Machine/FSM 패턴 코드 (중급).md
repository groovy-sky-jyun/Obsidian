## 상태 패턴을 활용한 캐릭터 이동
#### 1. 상태 인터페이스 정의
모든 상태 클래스가 구현해야 할 공통 인터페이스(추상 클래스)를 작성한다.
```cpp titile:IState hl:2
//전방 선언
class Character;
  
class IState {
public:
	virtual ~IState(){}
	virtual void OnEnter(Character* owner) = 0; //상태 진입 시 호출
	virtual void OnUpdate(Character* owner, float dt) = 0; // 상태 업데이트 시 호출
	virtual void OnExit(Character* owner) = 0; // 상태 빠져나갈 시 호출
	virtual void HandleInput(Character* owner, char input) = 0;
};
```

**OnEnter()**
- 상태에 진입할 때 한 번 호출된다. (초기화 로직)

**OnUpdate()**
- [[FSM 패턴 코드 (초급)]] Character 클래스의 Update()에 정의되어 있는 switch문문 로직이 여기로 들어온다.

**OnExit()**
- 다른 상태로 바뀌기 직전에 호출 (마무리/정리 로직)

#### 2. 상태 클래스 구현
```cpp title:StateClass
class IdleState : public IState{
public:
	void OnEnter(Character* owner) override {
		//플레이어 애니메이션을 Idle로 변경
	}
	void OnUpdate(Character* owner, float dt) override{
		//Idle 상태에서 매 프레임마다 할 일
	}
	void OnExit(Character* owner) override {
		//Idle 상태가 끝날 때 실행할 로직직
	}
	void HandleInput(Character* owner, char input) override {
		// 'w' -> Walking로 변환
		if(input == 'w'){
			owner->ChangeState(std::make_unique<WalkingState>());
		}
	}
};

class WalkingState : public IState{
public:
	void OnEnter(Character* owner) override {
		//플레이어 애니메이션을 Walk로 변경
	}
	void OnUpdate(Character* owner, float dt) override{
		//location += walkSpeed * dt;
	}
	void OnExit(Character* owner) override {
		//Walk 상태가 끝날 때 실행할 로직직
	}
	void HandleInput(Character* owner, char input) override {
		// 's' -> Idle로 변환
		if(input == 's'){
			owner->ChangeState(std::make_unique<IdleState>());
		} // 'r' -> Running로 변환
		else if(input == 'r'){
			owner->ChangeState(std::make_unique<RunningState>());
		}
	}
};

class RunningState : public IState{
public:
	void OnEnter(Character* owner) override {
		//플레이어 애니메이션을 Running으로 변경
	}
	void OnUpdate(Character* owner, float dt) override{
		if(stamina <=0){
			// 스태미나가 다 닳으면 강제로 Walking 상태로 변경		
		}
		else{
			//location += runSpeed * dt;
			stamina -= 10.0f * dt;
		}
	}
	void OnExit(Character* owner) override {
		//Running 상태가 끝날 때 실행할 로직
	}
	void HandleInput(Character* owner, char input) override {
		// 's' -> Idle
		if(input == 's'){
			owner->ChangeState(std::make_unique<IdleState>());
		}
	}

private:
	float stamina = 100.0f;
};
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