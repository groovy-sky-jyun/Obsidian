## 상태 객체화를 이용한 코드 구현

### 핵심 구조
각 상태를 별도의 클래스(객체)로 분리한다.

**1. <mark style="background: #ADCCFFA6;">상태 인터페이스</mark>**
- 모든 상태 클래스가 상속받을 인터페이스(추상 클래스)이다.
- **HandleInput(), Update(), Enter(), Exit() 같은 공통 함수를 정의**하여 일관성을 유지한다.

**2. <mark style="background: #ADCCFFA6;">각각의 상태 클래스</mark>**
- 구체적인 상태를 구현하는 클래스이다.
- 추상 함수들을 구현한다.
- 각 클래스는 해당 상태에 맞는 고유한 로직을 가지고 있다.
- <span style="color:rgb(255, 207, 61)">**HandleInput()**</span>
	- 상태 전이 규칙을 정의
- <span style="color:rgb(255, 207, 61)">**Enter()**</span>
	- 상태 진입 시 호출되는 함수
	- 해당 상태로 전환될 때 한 번만 실행되는 초기화 로직
- <span style="color:rgb(255, 207, 61)">**Update()**</span>
	- 상태 실행 로직을 처리하는 함수
	- 프레임마다 호출된다. 
- <span style="color:rgb(255, 207, 61)">**Exit()**</span>
	- 상태 이탈 시 호출되는 함수
	- 다른 상태로 전환될 때 한 번만 실행되는 마무리 로직

**3. <mark style="background: #ADCCFFA6;">상태를 가지는 주체</mark>**
- 플레이어나 몬스터가 해당된다.
- <span style="color:rgb(255, 207, 61)">**ChangeState(State* newState)**</span>
	- 상태를 전환하는 함수
	- 현재 상태의 Exit()를 호출하고, 새로운 상태의 Enter()를 호출한다.

---

#### 1. 상태 인터페이스 
```cpp title:IPlayerState hl:2
//전방 선언 
class Player;
  
class IPlayerState {
public:
	// 상태 진입 시 호출
	virtual void Enter(Player* player) = 0;
	// 키 입력에 따른 상태 전이 조건 확인
	virtual IPlayerState* HandleInput(Player* player, char key) = 0;
	// 상태 로직 실행
	virtual void Update(Player* player) = 0; 
	// 상태 빠져나갈 시 호출
	virtual void Exit(Player* player) = 0; 
	
	virtual ~IPlayerState(){}
};
```

<br>

#### 2. 상태 클래스
**Enter()**
- 해당 상태에서 한번만 실행될 로직

**HandleInput()**
- 매 프레임마다 확인할 상태 전이 규칙

**Update()**
- 매 프레임마다 실행될 로직


```cpp title:States hl:57
class IdleState : public IPlayerState{
public:  
	void Enter(Player* player) override {
		// Idle 상태 진입
	}
	IPlayerState* HandleInput(Player* player, char key) override{
		if(key == 'a'){
			player->SetMoveDirection(MoveDirection::Left);
			return new MoveState()
		} else if(key == 'd'){
			player->SetMoveDirection(MoveDirection::Right);
			return new MoveState()
		} else if(key == 'c'){
			return new JumpState();
		}
		return this;
	}
	void Update(Player* player) override {
		// Idle 애니메이션 실행
	}
	void Exit(Player* player) override {
		// Idle 상태 탈출
	}
};

class MoveState : public IPlayerState{
public:
	void Enter(Player* player) override {
		// Move 상태 진입
	}
	IPlayerState* HandleInput(Player* player, char key) override{
		if(key == 'a'){
			player->SetMoveDirection(MoveDirection::Left);
			return new MoveState()
		} else if(key == 'd'){
			player->SetMoveDirection(MoveDirection::Right);
			return new MoveState()
		} else if(key == 'c'){
			return new JumpState();
		}
		return new IdleState();
	}
	void Update(Player* player) override {
		MoveDirection direction = player->GetMoveDirection();
		if(direction == MoveDirection::Left){
			// 캐릭터 왼쪽으로 이동
		}
		else if(direction == MoveDirection::Right){
			// 캐릭터 오른쪽 이동
		}
	}
	void Exit(Player* player) override {
		// Move 상태 탈출
	}
};

class JumpState : public IPlayerState{
private:
	int jumpCounter = 3;
public:
	void Enter(Player* player) override {
		if(player && player->GetCharacterMovement()){
			player->Jump(); 
		}
	}
	IPlayerState* HandleInput(Player* player, char key) override{
		if(key == 'a'){
			player->SetMoveDirection(MoveDirection::Left);
			return this;
		} else if(key == 'd'){
			player->SetMoveDirection(MoveDirection::Right);
			return this;
		}
		return this;
	}
	void Update(Player* player) override {
		 if(player && player->GetCharacterMovement()){
			 if(!player->GetCharacterMovement()->IsFalling()){
				 //falling이 false면 착지했다는 뜻
				 player->ChangeState(new IdleState());
			 }
		 }
	}
	void Exit(Player* player) override {
		// Jump 상태 탈출
	}
};
```
##### JumpState
- Enter()
	- 점프는 JumpState에 진입해서 한번만 실행되어야 한다.
- Update()
	- 바닥에 착지했는지 확인한다.
- HandleInput()
	- 바닥에 착지했으면 자동으로 Idle상태로 바뀐다.
	- 즉, JumpState의 HandleInput()이 호출되었다는것은 아직 점프도중이여서 IdleState로 변경되지 않은 상태란 뜻이다.
	- 점프중일때는 다른 키를 입력해도 다른 상태로 변하지 않으므로 캐릭터가 바라보는 방향만 바꿔준다.

<br>

#### 3. Player
```cpp title:Player hl:32,43
enum MoveDirection{ 
	Left,
	Right,
	None
} 
class Player{ 
private:
	IPlayerState* currentState;
	MoveDirection moveDirection;

public:
	Player() : currentState(nullptr){
		ChangeState(new IdleState());
		moveDirection = MoveDirection::Right
	}
	~Player() {
		if(currentState){
			delete currentState;
		}
	}

	void SetMoveDirection(MoveDirection newDirection){
		moveDirection = newDirection;
		// 방향에 따른 플레이어 시선 변경
	}
	
	MoveDirection GetMoveDirection(){
		return moveDirection;
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
	
	void Tick(char key){
		IPlayerState* nextState = currentState->HandleInput(this, key);
		if(nextState != currentState){
			ChangeState(nextState);
		}
		currentState->Update(this);
	}
};
```
##### ChangeState()
1. `currentState->Exit(this);`
	- 현재 상태 탈출
2. `currentState = newState`
	- 현재 상태를 다음 상태로 변경
3. `currentState->Enter(this)`
	- 현재 상태 진입

##### Tick()
1. Tick(char key)
	- 현재 키 입력을 받는다.
2. `currentState->HandleInput(this,key);`
	- 현재 상태의 전이 규칙에 따라 다음 상태를 결정한다.
3. `ChangeState(nextState);`
	- 현재 상태와 다음 상태가 다르다면 상태를 변경한다.
4. `currentState->Update(this);`
	- 현재 상태 실행 로직을 호출한다.

<br>

#### 4. 실행
```cpp title:main
int main(){
	Player player;
	char key;
	while(true){
		std::cin >> key;
		
		if(key == 'q'){
			break;
		}
		
		player.Tick(key);
		
		//딜레이
		std::this_thread::sleep_for(std::chrono::milliseconds(50));
	}
	return 0;
}
```

---

#### 장점
- 각 클래스의 역할이 명확하게 분리되어 있어 유지보수성과 확장성이 높다.
	- `IdleState`는 '정지' 상태의 로직만 담당
	- `MoveState`는 '이동' 상태의 로직만 담당
	- `JumpState`는 '점프' 상태의 로직만 담당
	- `Player` 클래스는 상태를 전환하고, 각 상태에 공통적으로 필요한 데이터를 관리
- 새로운 상태를 추가/기존 상태 로직 변경 시, 해당 상태 클래스만 수정하면 된다.
- 상태를 객체로 분리하여 낮은 결합도와 높은 응집도를 가진다.


#### 주의사항
>##### 순환 참조 문제
>위의 코드를 구현할 때 Player 클래스에서 IState를 include 하고, 각각의 상태 클래스 또한 Player 클래스를 include 하게 되면 순환참조 문제가 발생할 수 있다.
>
>##### 해결 방법
>이를 해결 하기 위한 방법은 전방 선언을 사용하는 것이다.
>
>IState 클래스의 멤버 함수들은 Player 포인터만 매개변수로 받고 있다.
IState는 Character의 크기나 구체적인 멤버에 대해 알 필요가 없으므로 `#include "Charcter.h"` 대신 `class Character;` 라는 전방선언을 사용해준다. 

<br>

 > ##### #전방선언
> 헤더파일에서 특정 클래스의 포인터나 참조만 사용하면, 그 클래스의 헤더 파일을 include 할 필요 없이 전방선언으로 충분하다.
> 
> 하지만 그 클래스의 객체를 직접 생성하거나, 멤버에 접근, 크기를 알아야 할때는 반드시 헤더 파일을 include 해서 전체 정의를 알려주어야 한다.

---

#### 한계점
##### 1. 객체 생성 및 삭제의 비효율성
**문제점**
 - new와 delete 연산자를 사용해 상태 객체를 매번 동적으로 생성하고 소멸시킨다.

 **한계점**
-  빈번한 메모리 할당 및 해제는 성능 저하와 메모리 파편화를 유발할 수 있다.
 
##### 2. 복잡한 상태 전환의 한계
**문제점**
- 상태 전환 조건이 각 HandleInput 함수에 하드코딩되어있다. 

**한계점**
- 상태가 추가되면 각 상태마다 HandleInput을 수정해주어야 한다.
- 여러 상태가 복합적으로 작용하는 경우, HandleInput 함수가 매우 길고 복잡해진다.

#### 해결 방법
계층적 FSM을 사용하면 된다.
자세한 설명은 다음 페이지에 작성되어 있다.
[[HFSM 코드 (고급)]]