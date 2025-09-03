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

---

### 코드 구현
#### Monster
```cpp title:Monster hl:11,22
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
	
	// 상태 전이 규칙 정의 및 실행
	void Tick(){
		if(/*플레이어 감지*/){
			if(/*플레이어와의 거리가 일정거리 이하*/){
				ChangeState(make_unique<ComboState>());
			} else{
				ChangeState(make_unique<ChaseState>());
			}			
		} else if (/*플레이어가 도망감 || HP 일정 수치 이하*/){
			if(/*HP가 0이하*/){
				ChangeState(make_unique<DeathState>());
			}else{
				ChangeState(make_unique<PatrolState>());
			}
		}
		
		if(m_currentState){
			m_currentState->Update(this);
		}
	}
};
```
> ##### IMonsterState
- 모든 상태 객체가 가지는 인터페이스이다.
	- 모든 상태 객체가 동일한 형태의 기능을 가지게 된다.
- 모든 상태 객체가 하나의 타입으로 다뤄질 수 있도록 하는 <span style="color:rgb(255, 207, 61)">다형성</span>을 구현하기 위한 클래스이다.
	- 해당 클래스 포인터를 통해 어떤 상태 객체에라도 접근할 수 있다.

> ##### ChangeState
- <span style="color:rgb(255, 207, 61)">최상위 상태 전이 로직</span> 구현
	1. 현재 최상위 상태를 빠져나간다.
	2. 현재 최상위 상태를 다음 최상위 상태로 변경한다.
	3. 현재 최상위 상태에 진입한다.

> ##### Tick
- <span style="color:rgb(255, 207, 61)">최상위 상태 클래스 전이 규칙</span>을 정의한다.
- 하위 상태 클래스는 알지 못한다.
- 전이 규칙에 따라 최상위 상태를 전이하고, <span style="color:rgb(255, 207, 61)">매 프레임마다 로직을 실행</span>한다.

#### 최상위 상태 객체
```cpp title:MonsterState hl:5,15,20,32
class PatrolState : IMonsterState{
private:  
	unique_ptr<IMonsterState> m_currentSubState;
public:
	void ChangeSubState(unique_ptr<IMonsterState> newState){
		if(m_currentSubState){
			m_currentSubState->Exit(this);
		}
		m_currentSubState = newState;
		if(m_currentSubState){
			m_currentSubState->Enter(this);
		}
	}
	
	void Enter(Monster* monster) override {
		// PatrolState로 진입 시 Wander로 시작
		ChangeSubState(make_unique<WanderState>());
	}

	void Update(Monster* monster) override {
		if(/*순찰 완료*/){
			ChangeSubState(make_unique<IdleState>());
		} else if(/*대기 완료*/){
			ChangeSubState(make_unique<WanderState>());
		}
		
		if(m_currentSubState){
			m_currentSubState->Update(monster);
		}
	}
	
	void Exit(Monster* monster) override {
		if(m_currentSubState){
			m_currentSubState->Exit(monster);
		}
	}
};
```
> ##### ChangeSubState
- <span style="color:rgb(255, 207, 61)">자식 상태 전이 로직</span> 구현
	1. 현재 자식 상태를 빠져나간다.
	2. 현재 자식 상태를 다음 자식 상태로 변경한다.
	3. 현재 자식 상태에 진입한다.
	
> ##### Update
- <span style="color:rgb(255, 207, 61)">자식 상태 클래스 전이 규칙을 정의</span>한다.
- 전이 규칙에 따라 자식 상태를 전이하고, <span style="color:rgb(255, 207, 61)">매 프레임마다 로직을 실행</span>한다.

#### 자식 상태 객체
```cpp title:Wander hl:7,11,18,30,35,41
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

// 2초간 제자리 대기 상태
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
		if(/* 2초 지났으면*/){
			bIdleStatus = true;
		}
	}
	
	void Exit(Monster* monster) override {}	

	bool getIdleStatus() { return bIdleStatus; }
};
```
> ##### Update
- 상태 전이 규칙 없이 <span style="color:rgb(255, 207, 61)">온전히 행동과 관련된 실행 로직</span>만 구현한다.

---

### 한계점
Monster::ChangeSate와 최상위 상태클래스의 ChangeSubState의 내부 동작이 같아 **<span style="color:rgb(255, 207, 61)">코드 중복</span>**이 발생한다. 

그럼 최상위 상태 클래스의 ChangeSubState를 삭제하고 Monster의 ChangeState를 사용하면 되지 않나 생각할 수 있다.
Monster 클래스에서 자식 상태를 직접 다루게 되면, 역할이 명확하게 분리되지 않고, Monster의 책임이 커져 응집도가 낮아지고, 결합도가 높아지는 문제가 발생한다.

### 해결 방법
##### 1. 템플릿 헬퍼 함수
상태 전환 로직을 템플릿 함수로 분리하여 코드 중복을 제거할 수 있다.
```cpp title:Templete
template <typename TState>
void ChangeState(TState* state, unique_ptr<IMonsterState> newState){
	if(state->m_currentState){
		state->m_currentState->Exit(state);
	}
	state->m_currentState = move(newState);
	state->m_currentState->Enter(state);
}
```

##### 2. 별도의 StateMachine 클래스
상태 전이와 관련된 모든 책임을 StateMachine 클래스가 지게 하는 것이다. (분리)

Monster 클래스는 상태 전환 로직에 대해 알 필요가 없다.
대신, 자신이 소유한 <span style="color:rgb(255, 207, 61)">StateMachine 인스턴스에 모든 상태 관련 작업을 위임</span>한다.
(최상위 상태를 바꿔줘! 라고 명령만 하고, 내부적으로 어떻게 교체해야 하는지는 알 필요가 없는것이다.)

흡사 Player 클래스가 자신의 인벤토리 상태가 어떠한지 알 필요 없이 **모든 권한을 Inventory 클래스에 위임**한것과 같다. 

-> <span style="color:rgb(255, 207, 61)">위임을 통해 코드 결합도를 줄이고, 각 클래스의 책임을 더욱 명확히 하여 유지보수가 쉬워진다.</span>

>**StateMachine**
>- 상태 전환 로직, 현재 상태 업데이트 역할 수행
>
>**Monster**
>- StateMachine 인스턴스를 하나 가진다.
>- 해당 인스턴스는 최상위 상태를 관리한다.
>
>**PatrolState(최상위 상태)**
>- 각각 자신의 StateMachine 인스턴스를 하나씩 가진다.
>- 해당 인스턴스는 자식 상태를 관리한다.

```cpp title:StateMachine ,hl:1-23
class StateMachine{  
private:
	unique_ptr<IMonsterState> m_currentState;
	
public:
	void ChangeState(unique_ptr<IMonsterState> newState, Monster* monster){
		if(m_currentState){
			m_currentState->Exit(state);
		}
		m_currentState = move(newState);
		m_currentState->Enter(state);
	}
	
	void Update(Monster* monster){
		if(m_currentState){
			m_currentState->Update(monster);
		}
	}
	
	const IMonsterState* GetCurrentState() const{
		return m_currentState;
	}
};

class Monster{
private:
	StateMachine m_stateMachine;
	
public:
	void Tick(){
		// 전이 규칙 정의
		m_stateMachine.ChangeState(make_unique</*최상위상태*/>(),this);
		m_stateMachine.Update(this);
	}
};

class Patrol : IMonsterState{
private:
	StateMachine m_subStateMachine;
	
public:
	void Enter(Monster* monster) override {
		m_subStateMachine.ChangeState(make_unique<WanderState>(), monster);
	}	
	
	void Update(Monster* monster) override {
		// 전이 규칙
		m_subStateMachine.ChangeState(make_unique<IdleState>(), monster);
		// 전이 규칙
		m_subStateMachine.ChangeState(make_unique<WanderState>(), monster);
		
		m_subStateMachine.Update(monster);
	}
	
	void Exit(Monster* monster) override{
		// Patrol 탈출
	}
};
```
> ##### StateMachine을 포인터가 아닌 객체로 사용하는 이유?
> - 객체로 선언하면 stack에 할당되어, 해당 객체가 속한 객체의 <span style="color:rgb(255, 207, 61)">생명주기</span>와 함께 생성되고 소멸한다. (new, delete 사용 안하므로 메모리 누수 위험 제거)
> - 객체의 <span style="color:rgb(255, 207, 61)">소유권 관계</span>가 좀 더 명확해진다.
> - 포인터는 <span style="color:rgb(255, 207, 61)">null일 가능성</span>을 고려해야 하지만, 객체는 고려하지 않고 바로 접근할 수 있다.