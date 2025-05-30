세가지 패널로 구성된다.
1. 그래프 : 분기와 노드의 레이아웃을 시각적으로 구성
2. Details : 노드의 프로퍼티 정의
3. Black Board : Key Value 확인
---
### Composites
흐름 제어 형태, 분기 실행 방식을 결정한다.
노드의 흐름은 왼쪽에서 오른쪽으로 실행된다.

#### #Selector
__<span style="color:rgb(255, 192, 0)">동작 방식</span>__
자식 노드를 순차적으로 실행하며, 성공한 노드에서 이동을 멈춘다.

__성공 조건__
자식 노드 중 하나라도 성공하면 Selector 성공

__실패 조건__
모든 자식 노드가 실패하면 Selector도 실패

#### #Sequence
__<span style="color:rgb(255, 192, 0)">동작 방식</span>__
실패하는 노드를 만날때 까지 순차적으로 실행

__성공 조건__
모든 자식 노드가 성공이면 Sequence 성공 

__실패 조건__
자식 노드를 순차적으로 실행할 때, 하나라도 실패하면 중단

#### #SimpleParallel
__<span style="color:rgb(255, 192, 0)">동작 방식</span>__
Main Task, Background Branch 두 개의 작업을 동시에 수행한다.
1. __Main Task__
	- 완료 시 SimpleParallel 종료 
2. __Background Branch__
	- Main Task와 병렬로 실행 (Main Task가 종료되기 전까지)
	- Tick으로 호출된다. (반복 작업에 용이)

__성공 조건__
모든 자식 노드가 성공이면 Sequence 성공 

__실패 조건__
자식 노드를 순차적으로 실행할 때, 하나라도 실패하면 중단
- Main Task
- Background Branch

---
#### #Task
AI가 수행할 액션이다.

#### #Decorator
특정 노드가 실행될 조건이다.
이 조건의 값에 따라 해당 노드의 실행이 멈출 수 있다.

| Abort Type | Explain |
|---|---|
| None | 조건에 부합할 시 실행 x |
| Self | 현재 실행 중인 해당 노드 중단 |
| Lower Priority | 해당 노드보다 우선순위 낮은 것만 중단 |
| Both | 해당 노드 및 낮은 우선순위 전부 중단 |

#### #Service
- 노드가 실행 중일 때 반복적으로 실행된다. (Tick)
- 실시간으로 BlackBoard Key값을 체크하거나 업데이트 할 수 있다.

__실행 조건__
해당 Service가 붙은 노드가 활성화 되어 있어야 실행된다.

---

Event Receive Execute
Event Receive Abort
Event Receive Tick

