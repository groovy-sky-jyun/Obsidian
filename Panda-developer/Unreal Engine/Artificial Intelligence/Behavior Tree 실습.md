### 동작 과정
1. Agent가 바라보는 시야에 플레이어가 있는지 확인
2. 시야에 플레이어가 있다면 해당 플레이어 위치로 이동
	1. 시야에 플레이어가 있는 동안 2 반복
3. 시야에 플레이어가 없다면 반경 100 안에 랜덤 위치로 이동
	1. 시야에 플레이어가 없는 동안 3 반복

<span style="color:rgb(255, 82, 82)">(동영상) (동영상)</span>

### 구현 방법
#### Black Board
BlackBoard는 상태나 값을 담고 있는 Key들의 집합이다.
해당 기능을 구현하기 위해서 다음과 같은 키들을 생성해준다.
![[BB_Enemy.png|300]]

```cpp title:BB_Enemy
//Enemy가 바라보는 플레이어를 나타내는 객체
Actor* EnemyActor 
//Enemy의 시야에 플레이어가 들어왔는지 여부
bool HasLineOfSight
//순찰할 때의 랜덤 위치
Vector PatrolLocation
//Enemy 스스로를 가리키는 객체
Actor* SelfActor
```

#### AIControll_Enemy
##### AIPerception 추가
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

##### Event Graph
`On Target Perception Update`를 통해 감각기관이 Actor를 감지할 때마다 실행되는 함수를 생성한다.

1. Actor를 감지에 성공 & Actor가 Player라면
	1. Timer Handle을 초기화
	2. BlackBoard의 HasLineofSight 값을 true로 설정
	3. Enemy Actor를 감지한 Actor로 설정
2. 감지에 실패 | Actor가 Player가 아니라면 
	1. 4초 Timer Handle 생성
	2. 4초 뒤에도 감지에 실패한 상태라면
		1. BlackBoard의 HasLineofSight 값을 false로 설정
		2. Enemy Actor를 nullptr로 설정
	3. 4초가 되기 전, 감지에 성공하면 1.1로 돌아간다.

Timer Handle의 역할을 부가적으로 설명하겠다.
이 코드에서 Timer Handle을 사용하지 않아도 문제는 없다. 2.3을 좀 더 설명하자면 
![[AIController Event Graph body.png]]


우선은 AIControll을 통해서 Behavior Tree를 실행하도록 해야한다.

사전에 BT_Enemy(Behavior Tree)를 만들어 두고, 다음과 같이 BTAsset에 BT_Enemy를 설정해준다.

![[AIController Event Graph excute.png|400]]
![[AIController Event Graph.png]]