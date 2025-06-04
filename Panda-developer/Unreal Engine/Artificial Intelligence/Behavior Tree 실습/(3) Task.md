Task는 실행하는 업무(동작)을 나타낸다.
Behavior Tree에서 Task 노드에 도착하면 해당 동작을 실행한다.

여기에서는 Chase, Patrol 2가지 Task를 작성할 것이다.
이를 통해 <span style="color:rgb(255, 192, 0)">BlackBoard의 Key값 동기화 설정</span>을 해준다.

---
### Chase Player
Agent의 플레이어를 뒤쫓을 때 속도를 정의한다.

Finish Execute의 Success값은 해당 Task가 Behavoir Tree에서 성공인지 실패인지 알려주기 위함이다.

![[Pasted image 20250604183802.png]]

> **✔️** __TIP__
>Chase Speed값을 Behavior Tree에서도 지정할 수 있도록 `Details > Insatance Editable`을 true로 설정해준다.
>![[Pasted image 20250604190636.png|300]]

---

### Find Random Patrol
반경 이내 랜덤 위치로 이동(순찰)하는 Task이다.

Agent의 속도를 순찰할 때의 속도로 정의한다.

다음으로는 순찰 반경 이내에 랜덤 위치 여부에 따라 BlackBoard의 `PatrolLocation` Key값을 동기화 한다. 이때, GetRandomReachablePointRadius 함수를 이용한다.
- true: 랜덤 위치로 `PatrolLocation` 설정
- false: 현재 위치로 `PatrolLocation` 설정

![[Pasted image 20250604184933.png]]

> **✔️** __TIP__
>Patrol Speed, Patrol Radius, Patrol Location값을 Behavior Tree에서도 지정할 수 있도록 `Details > Insatance Editable`을 true로 설정해준다.
>![[Pasted image 20250604190851.png|300]]


