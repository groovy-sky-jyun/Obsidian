## 상태 객체화를 이용한 코드 구현

### 핵심 구조
각 상태를 별도의 클래스(객체)로 분리한다.

**1. <mark style="background: #ADCCFFA6;">상태 인터페이스</mark>**
- 모든 상태 클래스가 상속받을 인터페이스(추상 클래스)이다.
- Update(), Enter(), Exit() 같은 공통 함수를 정의하여 일관성을 유지한다.

**2. <mark style="background: #ADCCFFA6;">각각의 상태 클래스</mark>**
- 구체적인 상태를 구현하는 클래스이다.
- 추상 함수들을 구현한다.
- 각 클래스는 해당 상태에 맞는 고유한 로직을 가지고 있다.
- <span style="color:rgb(255, 207, 61)">Enter()</span>
	- 상태 진입 시 호출되는 함수
	- 해당 상태로 전환될 때 한 번만 실행되는 초기화 로직
- <span style="color:rgb(255, 207, 61)">Update()</span>
	- 상태 시행 로직을 처리하는 함수
	- 프레임마다 호출된다. 
	- **<span style="color:rgb(128, 202, 255)">상태 전이 규칙들을 정의한다.</span>**
- <span style="color:rgb(255, 207, 61)">Exit()</span>
	- 상태 이탈 시 호출되는 함수
	- 다른 상태로 전환될 때 한 번만 실행되는 마무리 로직

**3. <mark style="background: #ADCCFFA6;">상태를 가지는 주체</mark>**
- 플레이어나 몬스터가 해당된다.
- <span style="color:rgb(255, 207, 61)">ChangeState(State* newState)</span>
	- 상태를 전환하는 함수
	- 현재 상태의 Exit()를 호출하고, 새로운 상태의 Enter()를 호출한다.

---

#### 1. 상태 인터페이스 
```cpp title:IPlayerState hl:2
//전방 선언 
class Player;
  
class IPlayerState {
public:
	virtual ~IPlayerState(){}
	//상태 진입 시 호출
	virtual void Enter(Player* player) = 0;
	// 상태 업데이트 시 호출 
	virtual void Update(Player* player, char key) = 0; 
	// 상태 빠져나갈 시 호출
	virtual void Exit(Player* player) = 0; 
};
```

<br>

#### 2. 상태 클래스
```cpp title:States
class IdleState : public IPlayerState{
public:
	void Enter(Player* player) override {
		// Idle 상태 진입
	}
	void Update(Player* player, char key) override;
	void OnExit(Player* player) override {
		// Idle 상태 탈출
	}
};

class MoveState : public IPlayerState{
public:
	void Enter(Player* player) override {
		// Move 상태 진입
	}
	void Update(Player* player, char key) override;
	void Exit(Player* player) override {
		// Move 상태 탈출
	}
};

class JumpState : public IPlayerState{
public:
	void Enter(Player* player) override {
		// Jump 상태 진입
		jumpFrames = 3;
	}
	void Update(Player* player, char key) override;
	void Exit(Player* player) override {
		// Jump 상태 탈출
	}

private:
	int jumpFrames = 3; // 점프 지속 프레임 수
	float stamina = 100.0f;
};
```

<br>

#### 3. Player
```cpp title:Player hl:16,28
class Player{ 
private:
	IPlayerState* currentState;

public:
	Player() : currentState(nullptr){
		ChangeState(new IdleState());
	}
	
	~Player() {
		if(currentState){
			delete currentState;
		}
	}

	// 상태 전환 핵심 함수
	void ChangeState(IPlayerState* newState){
		if(currentState){
			currentState->Exit(this);
			delete currentState;
		}
		currentState = newState;
		if(currentState){
			currentState->Enter(this);
		}
	}
	
	void HandleInput(char key){
		if(currentState){
			currentState->Update(this, key);
		}
	}
};

void IdleState::Update(Player* player, char key){
	if(key == 'a' || key == 'd'){
		player->ChangeState(new MoveState());
		player>HandleInput(key);
	} else if(key == 'c'){
		player->ChangeState(new JumpState());
		player>HandleInput(key);
	}
}
void MoveState::Update(Player* player, char key){
	if(ket == 'a' || key == 'd'){
		//좌우 이동
	}else if(key == 'c'){
		player->ChangeState(new JumpState());
		player>HandleInput(key);
	}else{
		player->ChangeState(new IdleState());
		player>HandleInput(key);
	}
}
void JumpState::Update(Player* player, char key){
	if(stamina >= 10){
		stamina -= 10;
		jumpFrames--;
		if(jumpFrames <= 0){
			player->ChangeState(new IdleState());
			player>HandleInput(key);
		}
	}
}
```

##### 실행 흐름
1. 플레이어가 키를 입력
2. `Player::HandleInput()`이 실행
3. `currentState->Update()`를 통해서 currentState 클래스에서 전이 조건 확인
4. 전이 조건이 충족되면 `Player::ChangeState`를 호출한다.

**<span style="color:rgb(255, 82, 82)">문제점</span>**
```
player->ChangeState(new IdleState());
			player>HandleInput(key);
```
<span style="color:rgb(255, 82, 82)">이렇게가 맞나?
여기부터 다시 작성해야함(실행 코드는 제외)</span>

<br>

#### 4. 실행
```cpp title:main
int main(){
	Player player;
	char key;
	
	while(true){
		if(key == 'q'){
			break;
		}
		
		player.HandleInput(key);
		
		//딜레이
		std::this_thread::sleep_for(std::chrono::milliseconds(50));
	}

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