### 보스 몬스터 패턴 구조
#### 전체 구조
- 행동 선택 -> 행동 실행 -> 행동 정리
- 위의 구조를 반복한다.


#### 상세 구조 (BT + FSM)
1. <mark style="background: #ADCCFFA6;">입력 / 명령 수집</mark>
	- 플레이어의 입력을 큐에 기록한다.
	- AI가 환경(Perception, 거리, 각도 등)을 파악하여 BT 블랙보드에 반영
2. <mark style="background: #ADCCFFA6;">행동 선택 (Behavior Tree)</mark>
	- Decorator로 여러 행동들 중 조건을 만족 하는 우선순위가 가장 높은 액선을 선택한다.
	- BTTask에서 StartAttack() 같은 실행 트리거를 호출한다.
3. <mark style="background: #BBFABBA6;">FSM : Startup</mark>
	- 회전락, 이동락, 슈퍼아머 등 적용
	- 액션 시작 연출(몽타주, 카메라, VFX 등)
4. <mark style="background: #BBFABBA6;">FSM : Active</mark>
	- Anim Notify 또는 타이머로 Active 진입 타이밍 결정
	- Active 진입 시 히트박스 ON, 투사체 스폰, 트레이스 시작 등
5. <mark style="background: #BBFABBA6;">FSM : Active2 (판정 적용)</mark>
	- 충돌/트레이스가 히트박스와 접촉하면 데미지, 경직, 상태이상 등 계산 후 적용
	- 중복 타격 방지(1회)
6. <mark style="background: #BBFABBA6;">FSM : Recovery</mark>
	- 히트박스 OFF, 락/수퍼아머 해제 등
	- 쿨다운, 다음 행동 대기
7. <mark style="background: #ADCCFFA6;">완료 신호 & 다음 행동 선택</mark>
	- FSM이 None(Idle)로 복귀
	- BT가 다시 행동 선택(반복)

---

### FSM 파이프라인 예시
1. Startup - 0.6초
	- 회전락, 이동락, 슈퍼아머 적용
	- 보스 발 밑에 붉은 원이 점점 밝아지며 반경 300cm 표시
	- Anim: 상체를 구부리며 기를 모으는 모션
	- BT에 AttackDone = false 
2. Active - 0.2초
	- 히트박스 ON : 보스 주변 반경 350cm 원형 타격
	- Damage : 35
	- 근거리 피격 : 짧은 히트스턴 0.35초
	- 바깥쪽 가장자리 피격 : 약한 넉백
	- 후속 기믹 시드 : 적중 여부와 상관없이 확산 충격파 3링 예약
3. 확산 충격파
	- Active 종료 후 +0.25s/ +0.55s/ +0.85s에 링이 공중으로 던져짐 
	- 각 링은 얇은 빛을 띤다. (+3s 후 파괴)
	- 생성 시 0.05s 후 판정 ON, 3s 후 판정 OFF
	- 대응 수단: 링 사이 빈 공간으로 이동 또는 타이밍 맞춰 점프
4. Recovoery - 0.8초
	- 히트박스 OFF, 락/슈퍼아머 해제
	- 보스가 플레이어가 있는 방향으로 걸어간다.
	- 쿨다운 등록 : 5초
	- FSM은 None(Idle)로 복귀
	- BT에 AttackDone = true 

---






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