### enum + switch 이용한 캐릭터 이동

#### 1. 유한한 상태 종류
```cpp
enum State{
	Idle,
	Walking,
	Running
};
```
- player가 가질 수 있는 상태가 제한적으로 정의되어 있다.

#### 2. 전이 
```cpp
class Character{
public:
	void Update(float dt){
		switch(CurrentState){
		case State::Idle:
			//Idle 실행
			break;
		case State::Walking:
			//Walk (location += walkSpeed * dt) 실행
			break;
		case State::Running:
			//Running (location += runSpeed * dt) 실행행
			break;
		}
	}

	void HandleInput(char input){
		switch(CurrentState){
		case State::Idle:
			if(input == 'w') CurrentState = State::Walking;
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
- <span style="color:rgb(255, 192, 0)">현재 상태 + 입력</span>에 따라 다음 상태를 결정한다.

**Update**
- 현재 상태에 따른 동작 수행

#### 3. 입력에 따른 호출 (실행)
```cpp
int main(){
	Character player;
	player.Update(); //Idle 
	
	player.HandleInput('w'); //Idle -> Walking;
	player.Update();
	
	player.HandleInput('r'); //Walking -> Running;
	player.Update();
}
```
##### 주의 사항
>**Update()를 이렇게 작성하면 안된다.**
>
> 해당 코드는 쉬운 예시를 위함이지 실제로는 이렇게 작성해서는 안된다.
>위와같이 실행을 하면 한 프레임에 여러번의 Update 호출로 인해, 시간 적분, 물리, 애님 등에 문제가 발생할 수 있다.
>
> 예를 들어서 Walk() 함수 한 번 실행에 캐릭터가 앞으로 2cm 이동한다고 해보자. 이때, 한프레임 안에 플레이어가 Walk 키를 3번 입력하여 Update() 세번의 호출로 6cm가 이동했다. 그러면 사용자는 캐릭터가 갑자기 한프레임에 6cm나 이동한 것처럼 보이므로 끊긴것 처럼 보이게 된다.
>
>다른 예로는 한 프레임에 두번의 입력이 있을 때 플레이어에게 보여지는 애니메이션은 무조건 마지막에 입력된 상태의 애니메이션이 아니다. 프레임이 갱신될 때 눌려지는 키에 해당하는 애니메이션이 동작하게 되고, 한 프레임이 끝나기 전에 다른 키를 눌렀다 하더라도 애니메이션은 update되지 않는다. 하여 입력된 키 값과 맞지 않는 애니메이션이 실행될 수 있는 것이다.
>
>
>**그럼 어떻게 사용해야 하나?**
>프레임당 한 번만 Update되도록 해야한다.
>
>그러기 위해 메인 루프 또는 엔진의 Tick에서 Update(DeltaTime) 을 호출하도록 한다.
>