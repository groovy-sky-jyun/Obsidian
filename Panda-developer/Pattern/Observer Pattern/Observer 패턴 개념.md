### Observer 패턴의 목적
상태 변경 시, 이를 다수의 객체에게 알려야 할 때, 대상과 관찰자 간의 결합도를 최소화 하기 위해 Observer 패턴을 사용한다.

---

### Observer 패턴 사용 상황 예시
**체력 변화 감지 및 UI 갱신**
>대상 : HealthComponent
>관찰자 : HealthBarWidget, Damage Effect Spawner, 사망 처리 로직 등

<br>

**아이템 획득 및 인벤토리 업데이트**
>대상 : ItemPickupTrigger
>관찰자 : InventoryComponent, HUD, 퀘스트 시스템 등

<br>

**퀘스트 상태 변화**
>대상 : QuestManager
>관찰자 : 미니맵 마커, 퀘스트 로직(보상 지급) 등
