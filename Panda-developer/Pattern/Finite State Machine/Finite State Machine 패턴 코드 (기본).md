### 캐릭터 이동


--- 
#### 열거형
```cpp
enum State{
	Idle,
	Walking,
	Running
};
```
- player가 가질 수 있는 상태가 제한적으로 정의되어 있다.

```cpp
class Character{
public:
	void Update(float dt){
		switch(CurrentState){
		case State::Idle:
			//Idle
			break;
		case State::Walking:
			//Walk (location += walkSpeed * dt)
			break;
		case State::Running:
			//Running (location += runSpeed * dt)
			break;
		}
	}

	void HandleInput(char input){
		switch(CurrentState){
		case State::Idle:
			if(input == 'w') CurrentSstate = State::Walking;
			break;
		case State::Walking:
			if(input =='r') CurrentState = State::Running;
			else if (input =='s') CurrentState = State::Idle;
			break;
		case State::Running:
			if(Input =='s') CurrentState = State::Idle;
			break;
		}
	}
	
private:
	Stete CurrentState = Stete::Idle;
};
```
**HandleInput**
- 현재 상태에서 다음 상태 전의를 결정한다.
- 입력 값과 현재 상태를 바탕으로 다음 상태를 결정한다.

**Update**
- 현재 상태에 따른 동작 수행

```cpp
void Character::Tick(float DeltaSeconds){
	player.Update(DeltaSeconds);
}

void Character::MovingExample(){
	Character player;
	player.HandleInput('w'); //Idle -> Walking;
	player.HandleInput('r'); //Walking -> Running;
}
```
>**왜 HandleInput에서 Update를 호출하지 않고 main에서 따로 호출하는가?**
>
> 실무에서는 입력 처리와 갱신을 분리하는것이 표준적이다.
> 프레임 구조는 보통 `Input - Update - Render` 순서를 매 프레임 반복한다.


