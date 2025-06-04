Behavior Tree를 통해 AI가 어떤 동작들을 반복할 것인지를 정의해보자.

#### Root
AIRoot를 생성해준다. 이 노드가 실질적인 Root역할을 할 것이다. 
다음으로는 어떤 동작을 반복할것인지 설정한다. 
Agent는 다음과 같이 3개의 동작 중 1개의 동작을 무조건 실행할 것이다.
- Chase Player
- Patrol
- Wait

이 세개의 노드 중 한개라도 성공했다면 다시 AIRoot로 돌아가도록 AIRoot는 Selector로 생성한다.

![[Pasted image 20250604200653.png|500]]

---
#### Chase Player
플레이어를 쫓아가는 행동을 실행할 때는 시야에 감지됐는지 확인하는 조건이 필요하다. 이를 위해 `Mouse Right Click > Add Decorator > BlackBoard` 조건을 추가해준다. 

시야에 감지가 됐다면 해당 노드를 실행할 것이고, 감지가 되지 않았다면 Patrol을 실행할 것이다.


이때, Decorator의 Details는 다음과 같이 설정해준다.
- Observer aborts : Both
- Blackboard Key : HasLineofSight

![[Pasted image 20250604201304.png|500]]

> ##### Observer aborts 옵션에 따른 문제 상황
>  - __Self__
> __<span style="color:rgb(188, 149, 218)">상황</span>__
> Patrol이나 Wait 실행 중 HasLineofSight값 true로 변경
> __<span style="color:rgb(188, 149, 218)">문제</span>__
>  ChasePlayer가 실행되지 않는다. Patrol이나 Wait의 작업이 끝나고 다시 AIRoot로 돌아가면 실행된다.
>  <br>
>  - __Lower Priority__
>  __<span style="color:rgb(188, 149, 218)">상황</span>__
>   Chase Player 실행 중 HasLineofSight값 false로 변경
> __<span style="color:rgb(188, 149, 218)">문제</span>__
>  여전히 Chase Player가 실행 중이다. 해당 작업이 끝나고 다시 AIRoot로 돌아가면 Chase Player는 실행되지 않는다.
>  <br>
>  - __Both__
> __<span style="color:rgb(188, 149, 218)">해결</span>__
>  어떤 노드가 실행중이더라도 HasLineofSight값이 true, false 로 바뀜에 따라 원하는 대로 동작한다. (위의 두 상황에 대한 문제 해결)

<br>

##### 자식 트리
Chase Player의 자식 트리는 다음과 같은 동작을 순서대로 반복한다.
1. Player 방향으로 회전
2. 이동 속도 증가
3. Player 위치로 이동

![[Pasted image 20250604203153.png|500]]

이처럼 해당 동작들 전체를 순서대로 반복하다가 하나라도 실패가 생기면 AIRoot로 돌아가기 위해 Chase Player은 Sequence로 작성한다.

---
#### Patrol
##### 자식 트리
Patrol의 자식 트리는 다음과 같은 동작을 순서대로 반복한다.
1. 이동 속도 감소 및 랜덤 위치 설정
2. 랜덤 위치로 이동
3. 대기

![[Pasted image 20250604203730.png|500]]

이처럼 해당 동작들 전체를 순서대로 반복하다가 하나라도 실패가 생기면 AIRoot로 돌아가기 위해 Patrol은 Sequence로 작성한다.

---
#### 전체 Behavior Tree
![[Pasted image 20250604185826.png]]