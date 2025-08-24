### HFSM 이란?
FSM가 상태를 객체화 한 구조라면 HFSM은 이를 <span style="color:rgb(255, 207, 61)">계층 구조</span>로 관리하는 방식이다.
FSM에서는 상태 전이가 복잡해질수록 관리하기가 어려워진다.
이를 해결하기 위해, 부모-자식 관계로 상태를 만든다.

### 예시 상황 1
플레이어가 아래와 같은 상태를 가진다고 가정해보자.
- **최상위 상태** : PlayerState
- **자식 상태** : GroundState(IdleState, WalkState, RunState), AirState(JumpState)

![[Group 108 2.png|600]]

플레이어가 공격을 맞으면 데미지를 입는 로직은 Idle, Walk, Run 상태에서 동일하게 적용된다.

FSM으로 이를 구현하려면 각각 객체 클래스에 데미지 입는 로직을 구현해 줘야 한다. 그럼 코드 중복이 매우 많아지게 된다.

HFSM을 적용하면 부모 상태인 GroundState에 데미지를 입는 공통 로직을 작성해 상속으로 코드 중복없이 간단하게 구현할 수 있다.

### 예시 상황 2
좀 더 복잡한 상황을 생각해보자.

몬스터는 다음과 같은 행동규칙을 가진다
- (플레이어를 발견하기 전)에는 정해진 경로를 _**순찰**_합니다.
- (플레이어를 발견하면) _**쫓아갑니다**._
- (플레이어에게 충분히 가까이 가면) _**공격**_을 시작합니다.
- 공격하는 동안에도 (플레이어의 움직임)에 따라 _**회피**_하거나, 다른 _**스킬**_을 사용합니다.
- (플레이어의 시야에서 벗어나거나 HP가 낮아지면) 다시 _**순찰**_ 모드로 돌아갑니다.

이를 HFSM으로 구현하려면 몬스터는 다음과 같은 상태 계층 구조를 가지게 된다.

![[Group 113 1.png]]

### 코드 구현
```cpp title:Monster
class IMonsterState{
public:
	virtual void Enter(Monster* monster) = 0;
	virtual void Update(Monster* monster) = 0; 
	virtual void Exit(Monster* monster) = 0; 
}
class Monster{
private:
	unique_ptr<IMonsterState> m_currentState;
public:
	void ChangeState(unique_ptr<IMonsterState> newState){
		if(m_currentState){
			m_currentState->Exit(this);
		}
		m_currentState = newState;
		if(m_currentState){
			m_currentState->Enter(this);
		}
	}
	
	void Tick(){
		if(/*플레이어 감지*/){
			if(/*플레이어와의 거리가 일정거리 이하*/){
				ChangeState(make_unique<ComboState>());
			} else{
				ChangeState(make_unique<ChaseState>());
			}			
		} else if (/*플레이어가 도망감 || HP 일정 수치 이하*/){
			ChangeState(make_unique<PatrolState>());
		}
		
		if(m_currentState){
			m_currentState->Update(this);
		}
	}
};
```
##### Tick
- 상위 상태 클래스 전이 규칙만 정의한다.
- 하위 상태 클래스는 알지 못한다.

```cpp title:MonsterState
class PatrolState : IMonsterState{
private: 
	unique_ptr<IMonsterState> m_currentSubState;
public:
	void Enter(Monster* monster) override {
		// PatrolState로 진입 시 Wander로 시작
		m_currentSubState = make_unique<WanderState>();
		m_currentSubState->Enter(monster);
	}

	void Update(Monster* monster) override {
		m_currentSubState->Update(monster);

		if(/*순찰 완료*/){
			m_currentSubState->Exit(monster);
			m_currentSubState = make_unique<IdleState>();
			m_currentSubState->Enter(monster);
		} else if(/*대기 완료*/){
			m_currentSubState->Exit(monster);
			m_currentSubState = make_unique<WanderState>();
			m_currentSubState->Enter(monster);
		}
	}
	
	void Exit(Monster* monster) override {
		if(m_currentSubState){
			m_currentSubState->Exit(monster);
		}
	}
};
```

```cpp title:Wander
//특정 위치까지 이동
class WanderState : IMonsterState{
private:
	bool bReachStatus = false;
	
public:
	void Enter(Monster* monster) override {
		bReachStatus = false
	}
	
	void Update(Monster* monster) override {
		// 목표 위치로 이동
		if(/*목표 위치 도착*/){
			bReachStatus = true;
		}
	}
	
	void Exit(Monster* monster) override {}

	bool getReachStatus() { return bReachStatus; };
};
```

```cpp 
class IdleState : IMonsterState{
private:
	const int kIdleDuration = 2; //2초 대기
	bool bIdleStatus = false;
public:
	void Enter(Monster* monster) override {
		// 타이머 시작
		bIdleStatus = false;
	}
	
	void Update(Monster* monster) override {
		// 2초가 지났으면 상태 전이
		bIdleStatus = true;
	}
	
	void Exit(Monster* monster) override {}	

	bool getIdleStatus() { return bIdleStatus; }
};
```
