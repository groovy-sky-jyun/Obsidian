### 목적
<span style="color:rgb(255, 207, 61)">유한한 개수의 상태</span>가 있을 때, 특정 조건이나 이벤트에 따라 한 상태가 <span style="color:rgb(255, 207, 61)">다른 상태로 전이</span>되는것을 효과적으로 구현하기 위한 패턴이다. 

---

### 핵심 개념
**<span style="color:rgb(255, 207, 61)">유한한 상태</span>**
- 시스템이 가질 수 있는 상태를 유한한 개수로 정의한다.
- ex) Idle, Walking, Running, Startup, Active 등

**<span style="color:rgb(255, 207, 61)">상태 전이</span>**
- 한 상태에서 다른 상태로 넘어가는 규칙을 정의한다.
- 이 규칙은 특정 이벤트나 조건(키 입력, 충돌, 타이머 만료 등)에 의해 발동된다.

**<span style="color:rgb(255, 207, 61)">한번에 하나의 상태</span>**
- 시스템은 항상 하나의 상태만 활성화 한다.

---

### 장점
1. **높은 가독성 및 유지보수성**
	- <span style="color:rgb(255, 207, 61)">상태별로 로직을 나눠 작성</span>하므로 상태가 명확히 구분되어, 가독성이 높아진다.
	- 특정 상태를 수정하거나 추가할 때도 해당 상태의 코드만 수정하면 되므로 유지보수가 쉬워진다.
2. **코드 재사용성**
	- 상태를 독립적인 클래스로 구현함으로써, 비슷한 행동을 하는 <span style="color:rgb(255, 207, 61)">여러 객체가 공통적으로 상태를 재사용</span>할 수 있다.
3. **버그 발생률 감소**
	- 현재 상태에서 <span style="color:rgb(255, 207, 61)">규칙에 맞는 상태 전이만 허락</span>하므로 원치 않는 전이를 방지할 수 있다.

---

### 구현 방식
##### <mark style="background: #FFB86CA6;">enum + switch</mark>
- 간단한 상태 전환을 구현할 때 사용
- 상태 전이 조건이 복잡하지 않은 경우
- 상태 객체를 생성할 필요가 없어, 메모리 오버헤드가 적다.
- [[FSM 코드 (초급)|enum + switch 예제 페이지]]

##### <mark style="background: #FFB86CA6;">상태 객체</mark>
- 상태마다 독립적인 로직이 필요한 경우 (복잡한 로직)
- 상태 추가와 수정이 필요한 경우 (유지보수성)
- [[FSM 코드 (중급)|상태 객체 예제 페이지]]

##### <mark style="background: #FFB86CA6;">HFSM</mark>
- 복잡한 AI 및 플레이어 제어
	- 상태 안에 상태가 있는 경우(계층 구조_상속)
- 여러 상태에서 공통으로 전이되는 상태가 있는 경우, 코드의 중복을 최소화 하고자 할 때
- 대규모 프로젝트에 주로 사용된다.
- [[HFSM 코드 (고급)|HFSM 예제 페이지]]

---

### 사용 예시
##### <mark style="background: #CACFD9A6;">캐릭터 컨트롤</mark>
- 플레이어나 NPC가 여러 행동 상태를 가질 때 사용
- ex) Idle, Walk, Run, Attack, Die 등

##### <mark style="background: #CACFD9A6;">몬스터/AI 행동 패턴</mark>
**상태 **
- Patrol, Chase, Attack, FindCover

**전이**
- Patrol 중 플레이어 발견 시 : Patrol -> Chase
- Chase 중 일정 거리 내 플레이어가 들어왔을 시 : Chase -> Attack
- Attack 중 HP가 일정 쉬 이하로 떨어졌을 시 : Attack -> FindCover

##### <mark style="background: #CACFD9A6;">UI/UX</mark>
**상태**
- MainMenu, Game In HUD, Inventory, PauseMenu

**전이**
- MainMenu -> '새 게임' 버튼 -> Game In HUD
- Game In HUD -> ESC 키 -> PauseMenu

##### <mark style="background: #CACFD9A6;">게임 시스템 흐름 제어</mark>
**상태**
- 시작 로딩, 레벨 로딩, 게임 진행 중, 게임 종료

**전이**
- 시작 로딩 완료 시 : 시작 로딩 -> 레벨 로딩
- 모든 레벨 로딩 완료 시 : 레벨 로딩 -> 게임 진행
- 플레이어 패배 시 : 게임 진행 -> 종료

