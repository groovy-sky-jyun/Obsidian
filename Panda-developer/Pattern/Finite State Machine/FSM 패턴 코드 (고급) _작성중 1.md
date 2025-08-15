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

```cpp title:IState
class Character; //전방선언
class IState{
public:
	virtual ~IState() = default;
	virtual void OnEnter(Character* owner) = 0;
	virtual void OnUpdate(Character* owner, float dt) = 0;
	virtual void OnExit(Character* owner) = 0;
};
```

```cpp title:SuperState
class SuperState : IState{
protected:
	std:unique_ptr<IState> mCurrentSubState;
public:
	//하위 상태 설정
	void SetSubState(std::unique_ptr<IState> newSubState, Character* owner){
		if(mCurrentSubState){
			mCurrentSubState->OnExit(owner);
		}
		mCurrentState = std::move(newSubState);
		if(mCurrentSubState){
			mCurrentSubState->OnEnter(owner);
		}
	}
};
```


```cpp title:PatrolState
class PatrolState : public SuperState{
public:
	void OnEnter(Character* owner) override{
		// 순찰 상태로 전이 시, Idle로 초기화
		SetSubState(std::unique_ptr<IdleState>(), owner); 
	}
	void OnUpdate(Character* owner, float dt) override{
		// 반경 100cm 이내 플레이어가 있으면 ChaseState로 전환
		if(owner->IsPlayerInRange(100.0f)){
			owner->ChangeState(std::make_unique<ChaseState>());
			return;
		}

		// 공통 로직 이외의 로직은 하위 상태에 위임
		if(mCurrentSubState){
			mCurrentSubState->OnUpdate(owner, dt);
		}
	}
	void OnExit(Character* owner) override {}
};
```

```cpp title:IdleState
class IdleState : public IState{
public:
	void OnEnter(Character* owner) override {
		/* 애니메이션 초기화 */
	}

	void OnUpdate(Character* owner, float dt) override {
		//2초 타이머 생성
		if(/*타이머 2초 지났으면*/){
			owner->SetCurrentState(std::make_unique<WalkState>());
		}
	}
	void OnExit(Character* owner) override {}

}
```