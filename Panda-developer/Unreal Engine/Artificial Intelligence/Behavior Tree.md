### <span style="color:rgb(188, 149, 218)">Behavior Tree</span>
BT는 행동의 우선순위를 기반으로 트리를 순회하며 수행할 행동을 결정한다.

항상 Root 노드에서 시작하며, 위에서 아래로, 왼쪽에서 오른쪽으로 진행된다. (우선순위)



---
### <span style="color:rgb(188, 149, 218)">Blackboard</span>
AI의 뇌 역할을 한다. AI가 최종적으로 진행할 행동을 결정할 때 필요한 모든 데이터를 키와 값의 쌍으로 관리한다.

---
### <span style="color:rgb(188, 149, 218)">Composite</span>
흐름을 제어하는 노드이다.

> ##### <span style="color:rgb(255, 192, 0)">Selector</span>
> 자식 노드들 중 하나가 성공될 때까지 순차적으로 노드를 실행한다.
>
> 여러 행동 대안 중 하나를 선택해야 할 때 이상적이다. (공격, 추격, 순찰 등)
> 
> ##### <span style="color:rgb(255, 192, 0)">Sequence</span>
> 자식 노드들 중 하나가 실패할 때까지 순차적으로 노드를 실행한다.
> 
> 순서대로 완수해야 하는 일련의 행동에 적합(순찰 지점 찾기 -> 해당 지점으로 이동 -> 잠시 대기)
> 
> ##### <span style="color:rgb(255, 192, 0)">Simple Parallel</span>
> 두 개의 작업을 동시에 실행할 수 있다.
> 메인 Task가 실행되는동안 서브로 같이 실행되는 Task를 설정할 수 있다. (반드시 2개의 자식을 가진다.)
> (메인은 적을 향해 이동, 서브로 공격 애니메이션 재생)
> 1. __Main Task__
>	- 완료 시 SimpleParallel 종료 
> 2. __Background Branch__
>	- Main Task와 병렬로 실행 (Main Task가 종료되기 전까지)
>	- Tick으로 호출된다. (반복 작업에 용이)

---

### <span style="color:rgb(188, 149, 218)">Task</span>
실제 행동을 수행하는 작은 단위이다. 

Task는 반드시 성공/실패 결과를 반환해야 한다. 그래야 다음 행동 흐름을 부모 노드가 결정할 수 있다.
> __대표적인 Built-in Task__
> Move To : 지정 위치로 이동
> Wait : BT를 일시 정지
> Play Sound : 사운드 에셋 재생
> Rotate to face BB Entry : 지정 위치로 AI 회전


---

### <span style="color:rgb(188, 149, 218)">Decorator</span>
조건에 따라 특정 노드의 실행 여부를 결정하는 조건문 역할을 한다. 이 조건 값에 따라 해당 노드가 실행될 수도 멈출 수도 있다.

Composite Node, Task Node에 부착하여 해당 노드를 실행할지 결정한다.

조건문이 true여야 해당 노드와 하위 트리가 실행될 수 있다.

언제 조건문을 체크할건지 관찰자 중단 속성을 지정할 수 있다. Decorator가 제공하는 옵션은 다음과 같다.

| Observer Aborts | Explain |
|---|---|
| None | 해당 노드가 실행될 때만 조건 검사 |
| Self | 해당 노드의 조건이 바뀌면 자기 자신과 그 하위 트리의 실행이 영향을 받는다. |
| Lower Priority | 해당 노드보다 우선순위 낮은 것만 중단 |
| Both | 해당 노드 및 낮은 우선순위 전부 중단 |

> __주요 Decorator 종류__
> 
> *Blackboard Based*
> : Blackboard Key값을 기반으로 조건을 검사한다.
> 
> *Compoare BBEntries*
> : 두 개의 Blackboard Key값을 기반으로 조건을 검사한다. 
> 
> *Cooldown*
> : 해당 노드가 실행된 후, 지정된 시간 동안 다시 실행되지 않도록 막는다.
> 
> *Does Path Exist* 
> : 목표지점까지 유효한 NavMesh 경로가 존재하는지 확인한다.

---

### <span style="color:rgb(188, 149, 218)">Service</span>
특정 주기로 실행되며(Tick와 유사), AI 상태 업데이트 및 검사에 사용된다.
주기적으로 환경을 감지할 수 있다.
실시간으로 Blackboard Key값을 체크하거나 업데이트 할 수 있다.
Tick와 비슷하지만 다르게 사용할 수 있다.

---

Event Receive Execute
Event Receive Abort
Event Receive Tick

