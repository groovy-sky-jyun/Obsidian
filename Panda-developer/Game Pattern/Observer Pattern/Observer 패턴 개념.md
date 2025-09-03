### Observer 패턴의 목적
상태 변경 시, 이를 다수의 객체에게 알려야 할 때, 대상과 관찰자 간의 결합도를 최소화 하여 확장성과 유지보수성을 높이기 위해 Observer 패턴을 사용한다.

---

### 핵심 개념
**Subject**
>- 상태가 바뀌면 <span style="color:rgb(255, 192, 0)">알림을 보낸다.</span>

**Observer**
>- 원하는 Subject를 <span style="color:rgb(255, 192, 0)">구독</span>하고, 알림을 받으면 특정 이벤트를 실행한다.

**구독/해지**
>- Add / <span style="color:rgb(255, 192, 0)">Remove</span>를 통해 해지까지 포함하여 설계해준다.

**동기 호출**
>- 알림 시점에 관찰자 콜백이 순차적으로 실행되므로 해당 동작이 끝나기 전에 다른 동작을 할 수 없다.
>- 비동기 전환이 필요하다면 큐/타이머/스레드를 사용하여 분산 시킨다.

---

### Observer 패턴 시나리오
**체력 변화 -> UI/사운드/진동**
>- 대상 : HealthComponent
>- 관찰자 : HealthBarWidget, Damage Effect Spawner, 사망 처리 로직 등

**아이템 획득 -> 인벤토리/퀘스트/HUD**
>- 대상 : Pickup/Inventory
>- 관찰자 : Inventory UI, 퀘스트 시스템 등

**퀘스트 상태 변화 -> 미니맵/보상/로그**
>- 대상 : QuestManager
>- 관찰자 : 미니맵 마커, 퀘스트 로직(보상 지급) 등
