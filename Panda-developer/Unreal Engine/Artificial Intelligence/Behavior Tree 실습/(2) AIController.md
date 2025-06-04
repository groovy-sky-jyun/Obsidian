### AIPerception 추가
1. Enemy의 시야에 감지되는 것이 생길 때마다 함수를 실행하기 위해선 Enemy에 감각 기관을 설정해주어야 한다.
`Components > +Add > AIPerception` 를 통해 감각기관을 추가해준다.

2. Enemy에게는 시야가 필요하므로 AIPerception에 Sight를 등록해주어야 한다. `Components AIPerception > Details > Senses Config > + > Idex[0]` 를 통해서 `AI Sight config`를 등록해준다.

3. `AI Sight config > Sense > Detection by Affiliation > Detect Nutrals`를 true로 설정해준다.
	> __Detection by Affiliation__
	> 적/우호/중립 중 어떤 Actor를 감지할 지 Team ID를 기준으로 설정한다.
	> - Detect Enemies : AI가 적으로 판단하는 대상 
	> - Detect Friendlies : AI와 같은 Team ID를 가진 대상 (팀 기반 행동)
	> - Detect Neutrals : Team ID가 설정되지 않은 경우 (중립)
	

![[AIController Components.png|400]]

---

### Event Graph
`On Target Perception Update`를 통해 감각기관이 Actor를 감지에 변화가 생겼을 때마다 실행되는 함수를 생성한다. Actor가 감지되었을때와 놓쳐서 감지 대상이 사라졌을 때 동작한다.

이를 이용하여 감지 여부와 대상을 가리키는 BlackBoard의 `HasLineofSight`, `EnemyActor` Key값을 동기화 시켜줄 것이다.

__1. (감지에 성공했는지 & 그 대상이 Player인지) 확인__
- 감지된 Actor 목록 중 Tag가 Player인 Actor를 가져온다.
- 감지가 된것인지, 놓쳐서 감지대상이 사라진 것인지 확인하기 위해 Successfully Sensed 값을 받는다.
![[Pasted image 20250604180757.png]]

__2. 감지 성공 & 대상이 Player__
1. Timer Handle 제거
	- 타이머가 끝나기 전에 대상을 찾았으므로 타이머를 삭제해준다.
	- (플레이어를 4초 동안 못 찾으면 랜덤 위치로 이동)
2. BlackBoard의 `HasLineofSight = true`
3. BlackBoard의 `EnemyActor = 감지한 Actor`
![[Pasted image 20250604180831.png]]

__3. 감지 실패 | 대상이 !Player__
1. 4초 Timer Handle 생성	
2. 4초 뒤에도 감지에 실패한 상태라면7
	1. `SetTimerbyEvent` 함수 실행
		1. BlackBoard의 `HasLineofSight = false`
		2. BlackBoard의 `EnemyActor = nullptr`
3. 4초가 되기 전, (1)의 조건이 만족되면 (2)가 실행된다.
	- (2)가 실행되면서 생성된 Timer Handle이 제거되기 때문에 `SetTimerbyEvent`는 실행되지 않는다.
![[Pasted image 20250604180932.png]]

#### 전체 사진
Agent가 BehaviorTree를 바로 실행하도록 Event On Prossess와 Run Behavior Tree 노드를 연결시켜준다.
![[Pasted image 20250604204754.png|400]]
![[Group 68.png]]