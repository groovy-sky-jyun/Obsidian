### 명령 패턴 3가지 구성 요소
1. **요청자 Invoker**
	- 명령을 보관, 실행하는 역할을 하며, 명령의 내부 구현에 대해 알 필요가 없다.
	- 명령의 실행 시점을 결정한다.
2. **명령 객체 Command**
	- 실행할 행동을 캡슐화한 객체 (행동 자체를 객체로 만듬)
3. **수신자 Receiver**
	- 명령을 전달받아 실제로 작업을 수행하는 대상 

**예시**
1. 플레이어가 키보드를 누른다. -> Invoker(Input System)
2. F키에는 JumpCommand가 바인딩 되어 있다. -> Command (JumpCommand)
3. 점프 명령이 Player 객체의 jump()를 호출한다. -> Receiver(Player)

---

### Command Pattern 적용 
플레이어의 입력에 따라 특정 행동을 실행하는 코드를 구현해보자.

#### <span style="color:rgb(128, 202, 255)">단순한 입력 처리 코드</span>
```cpp title:code1
void InputHandler::handleInput(){
	if(isPressed(BUTTON_X)) jump();
	else if(isPressed(BUTTON_Y)) fireGun();
	else if(isPressed(BUTTON_A)) swapWeapon();
	else if(isPressed(BUTTON_B)) lurchIneffectively();
}
```
- 키와 행동을 직접 연결
- 행동 로직과 입력 처리 로직이 결합되어 있다.
- 입력을 추가/수정 할 경우, 행동을 교체할 경우 코드를 InputHandler 코드를 수정해야 한다.

#### <span style="color:rgb(128, 202, 255)">Command 패턴 사용</span>
여기에 Command 패턴을 적용하면 Invoker, Command, Receiver를 분리할 수 있다.

- Invoker : InputHandler -> 입력 받고 명령 실행
- Command : JumpCommand, FireCommand -> 수행할 행동을 정의
- Receiver : Actor : 실제로 행동을 수행하는 주체

##### 1단계: 명령 객체화
```cpp title:code2
class Command {
public:
	virtual ~Command(){}
	virtual void execute()=0;
};

class JumpCommand : public Command{
public:
	virtual void execute() {jump();}
};

class FireCommand : public Command{
public:
	virtual void execute() {fireGun();}
};
```
- Command는 추상 인터페이스이다.
- JumpCommand, FireCommand 등 각 명령 클래스는 행동을 정의한다.

>**문제점**
>명령 객체가 수행할 대상(Receiver)을 내부에서 직접 호출하고 있다.
>
> 즉, Command와 Receiver가 강하게 결합되어 있어, 다양한 대상에게 재사용하기 어렵다.

<br>

##### 2단계: Receiver 분리 - 실행 주체를 외부로 부터 주입
문제점을 해결하기 위해 Receiver를 분리해보자. 
```cpp title:code3
class Command {
public:
	virtual ~Command(){}
	virtual void execute(Actor* a) = 0;
};

class JumpCommand : public Command{
public:
	virtual void execute(Actor* a) { a->jump(); }
};
```
- 명령 객체는 행동을 정의
- 누가 행동을 수행할지는 외부에서 주입 (명령이 주체를 결정하지 않는다.)

<br>

**코드2와 코드3의 차이**
> 코드2 : `누가`행동할지와 `무엇을`할지가 명령 객체 내부에 고정되어 있다.
> 코드3 : `누가`는 외부가 정하고, `무엇을`할지는 명령이 정의한다.

<br>

**코드3의 확장성**
> 코드3이 <span style="color:rgb(255, 192, 0)">하나의 명령객체를 여러 대상에게 재사용</span>할 수 있기 때문에, 코드2 보다 **재사용성과 유연성이 높다.**
```cpp
JumpCommand* jumpCmd;

jumpCmd.execute(&Player);
jumpCmd.execute(&Enemy);
jumpCmd.execute(&Npc);
```