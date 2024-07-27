<br>

### 2. #KeyInput #LifeSpan
2번을 눌렀을 때 바로 파괴되는 것이 아닌 특정 시간이 지나면 파괴되도록 Life Span을 줘보도록 하겠다. 

#### (1) Life Span
어려울 것 없다. [[Key Input (1)_ Actor 생성 및 파괴]] 의 1-(5)에서 Destroy  노드만 변경해 줄 것이다. 

`Set Life Span`노드를 추가해 준 뒤 이 액터의 `In Lifespan`(생명시간)을 지정해준다. 여기서는 2초뒤 파괴되도록 2초를 지정해 준다. 
![[Pasted image 20240721125643.png]]

<br>

#### (2) 전체 blueprint
2번키를 눌렀을 때 지정된 시간이 지나고 파괴되는 기능을 구현한 event graph는 다음과 같다.
![[Pasted image 20240721125714.png]]
