### enum + switch 이용한 코드 구현
#### 1. 유한한 상태
```cpp
enum State{
	Idle,
	Walking,
	Running
};
```

#### 2. 전이 
```cpp hl:3,20
class Character{
public: 
	// 상태별 실행 로직 구현
	void Update(float dt){
		switch(CurrentState){
		case State::Idle:
			// Idle Animation 실행
			break;
		case State::Walking:
			// Walk Animation 실행
			// location += walkSpeed * dt
			break;
		case State::Running:
			// Running Animation 실행 
			// location += runSpeed * dt
			break;
		}
	}

	// 전이 규칙 정의 및 상태 전이
	void HandleInput(char input){
		switch(CurrentState){
		case State::Idle:
			if(input == 'w'){
				CurrentState = State::Walking;
			}
			break;
		case State::Walking:
			if(input =='r'){
				CurrentState = State::Running;
			}else if (input =='s'){
				CurrentState = State::Idle;
			}
			break;
		case State::Running:
			if(Input =='s'){
				CurrentState = State::Idle;
			}
			break;
		}
	}
	
private:
	State CurrentState = Stete::Idle;
};
```


#### 3. 입력에 따른 호출 (실행)
```cpp
int main(){
	Character player;
	player.Update(); //Idle 실행
	
	player.HandleInput('w'); //상태 전이 : Idle -> Walking;
	player.Update(); // Walking 실행
	
	player.HandleInput('r'); //상태 전이 : Walking -> Running;
	player.Update(); //행
}
```

--- 
#### update() 주의 사항
위의 코드는 쉬운 예시를 위함이지 실제로는 이렇게 작성해서는 안된다.
위와 같이 실행을 하면 <span style="color:rgb(255, 207, 61)">한 프레임에 여러 번의 상태 변경 로직 실행</span>으로 인해 시간 적분, 물리, 애님 등에 문제가 발생할 수 있다. 다음은 발생할 수 있는 예시들이다.

> **상황**
> : 한 프레임에 걷기, 점프 입력을 순서대로 한다.
> 
> **발생하는 문제**
> : 한 프레임이 끝났을 때의 캐릭터 상태는 2cm 이동된 위치에서 점프를 하고있다. 이때 2cm 이동하는 애니메이션은 보여지지 않고, 마지막 입력인 점프에 관한 애니메이션만 보여진다. 그러면 캐릭터가 순간이동한것처럼 보여진다. 

<br>

#### 해결 방법
- 프레임당 한 번만 Update되도록 Tick에서 Update()를 호출한다.
- HandleInput()에서 상태를 직접 바꾸는 대신, 델리게이트 같은 이벤트를 발생시킨다.
- 커맨트 패턴을 이용하여 상태를 큐에 쌓고 사용한다.

---

### enum + switch 한계점
위의 코드는 상태3개를 다루지만 상태가 30개가 된다면 어떨까?

**_유지보수성과 가독성 저하_**
- switch문이 굉장히 거대해져 <span style="color:rgb(255, 192, 0)">가독성이 떨어지게</span> 된다.
- 또한 새로운 <span style="color:rgb(255, 192, 0)">상태를 추가할때 마다 switch문을 수정</span>해야 하므로 버그가 생기기 쉽다.

**_불필요한 메모리 차지 및 응집도 저하_**
- 위의 코드처럼 Character 클래스가 모든 상태의 데이터를 한꺼번에 관리하면, 특정 상태에서만 필요한 데이터가 다른 상태에서도 <span style="color:rgb(255, 192, 0)">불필요하게 메모리를 차지</span>하게 된다.
- 예시 : Idle 상태일 때, Running 상태에서만 사용하는 Stamina 변수가 아무런 역할 없이 메모리 공간을 차지하고 있다.

**_상태 진입/탈출 로직 관리 어려움_**
- Update() 함수는 매 프레임 호출되므로, 상태 진입 시(OnEnter) 또는 탈출 시(OnExit)에만 실행되어야 하는 로직을 추가하면 원하는 대로 작동하지 않는다.
- 한번만 실행되어야 하는 로직이 매 프레임 실행될 것이다.

<br>

### 해결 방법
상태 패턴을 활용하여 각 상태를 독립적인 클래스로 만들면 객체 지향적으로 문제를 해결 할 수 있다. 

해당 코드는 아래 페이지에 작성되어 있다.
[[FSM 패턴 코드 (중급)]]