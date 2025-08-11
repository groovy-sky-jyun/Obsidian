Standing <-> Ducking
Standing <-> Jumping

```cpp 
enum ImageID{
	IMAGE_STAND,
	IMAGE_DUCK,
	IMAGE_JUMP
},

enum class Input{
	PRESS_DOWN,
	RELEASE_DOWN,
	PRESS_JUMP,
	LAND //점프 후 착지
}

//전방 선언
```
- 리소스 / 입력 정의

```cpp title:HeroineState
//상태 베이스 클래스
class HeroineState{
public:
	virtual ~HeroineState() = default;

	//전이가 필요한 경우 새 상태를 new 해서 반환
	virtual HeroineState* handleInput(Heroine& heroine, Input input){
		return nullptr;
	}

	//매 프레임 갱신
	virtual void update(Heroine& heroine, float dt){}

	//상태 진입
	virtual void enter(Heroine& heroine){}
};
```

```cpp title:Heroine
class Heroine{
public:
	Heroine(){
		state_ = new StandingState();
		state_->enter(*this);
	};
	
	~Heroine(){
		delete state_;
	};

	void handleInput(Input input){
		HeroineState* next = state_->handleInput(*this, input);
		if(next != nullptr){
			delete state_;
			state_ = next;
			state_->enter(*this);
		}
	};
	
	void update(float dt){
		state_->update(*this, dt);
	};

	void SetGraphics(Image img){
		CurrentImage = img;
		switch(img){
			case IMAGE_STAND:
				//render, method
				break;
			case IMAGE_DUCK:
				//render, method
				break;
			case IMAGE_JUMP:
				//render, method
				break;
		}
	}
	
private:
	friend class StandingState;
	friend class DuckingState;
	friend class JumpingState;
	
	HeroineState* state_ = nullptr;
	ImageId CurrentImage = IMAGE_STAND;
};
```

```cpp title:StandingState/DuckingState/JumpingState
class StandingState : public HeroineState{ 
public: 
	void enter(Heroine& heroine) override{
		heroine.setGraphics(IMAGE_STAND);
	}
	
	HeroineState* handleInput(Heroine& heroine, Input input) override{
	
	};
};

class DuckingState : public HeroineState{ 
public: 
	void enter(Heroine& heroine) override{
		heroine.setGraphics(IMAGE_DUCK);
	}
	
	HeroineState* handleInput(Heroine& heroine, Input input) override;
};

class JumpingState : public HeroineState{ 
public: 
	void enter(Heroine& heroine) override{
		heroine.setGraphics(IMAGE_JUMP);
	}
	
	HeroineState* handleInput(Heroine& heroine, Input input) override;
};
```




---



```cpp title:StandingState
class StandingState : public HeroineState {
public:  
	virtual void enter(Heroine& heroine){
		heroine.setGraphics(IMAGE_STAND);
	}
}
```

```cpp title:Heroine
void Heroine::handleInput(Input input){
	HeroineState* state = state_->handleInput(*this, input);
	if(state != NULL){
		delete state_;
		state_ = state;

		state_->enter(*this); //새로운 상태 입장
	}
}
```

```cpp title:DuckingState
HeroineState* DuckingState::handleInput(Heroine& heroine, Input input){
	if(input == RELEASE_DOWN){
		return new StandingState();
	}
}
```