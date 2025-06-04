### 동작 과정
1. Agent가 바라보는 시야에 플레이어가 있는지 확인
2. 시야에 플레이어가 있다면 해당 플레이어 위치로 이동
	1. 시야에 플레이어가 있는 동안 2 반복
3. 시야에 플레이어가 없다면 반경 100 안에 랜덤 위치로 이동
	1. 시야에 플레이어가 없는 동안 3 반복

<span style="color:rgb(255, 82, 82)">(동영상) (동영상)</span>

--- 
## 구현 방법
### Black Board
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