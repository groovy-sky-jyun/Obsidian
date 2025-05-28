AI가 경로 찾기를 통해 이동할 수 있도록 범위를 지정해준다.
그럼 범위 내에서 시작 위치와 목적지 사이 경로를 찾는다.

### #NavMesh
Navigation Mesh를 줄여서 Nav Mesh라고 한다.
World Collision Geometry에서 생성되며, PolygonMesh를 통해 레벨에서 이동 가능한 공간을 나타낸다. <span style="color:rgb(125, 125, 125)">(Nav Mesh를 타일로 분할할 수 있다.)</span>

### 비용 Cost
Nav Mesh를 사용하여 한 지점에서 다른 지점으로 이동할 때, 지나가는 경로상의 모든 영역 비용의 합을 뜻한다. 

<span style="color:rgb(188, 149, 218)">AI는 항상 비용이 가장 낮은 경로를 찾아 이동</span>하기 때문에 영역의 비용을 조절하여 동선을 수정할 수 있다.

예를 들어 거친 지형에는 AI가 오지 못하도록 영역 비용을 높여버리면 AI가 찾은 비용이 가장 낮은 경로에 거친 지형이 포함될 확률이 낮아지므로 해당 영역을 피할 수 있다.

그러나 거친 지형을 포함하는것이 경로 상 비용이 제일 낮다면 비용이 높은 영역이라 하더라도 지날 수 있다.

---
### Navigation Mesh Bounds Volume
레벨에서 Navigation을 생성할 영역을 지정한다. 

Place Actors 패널에서 Navigation Mesh Bounds Volume 을 찾아 레벨로 드래그 하면 영역을 지정할 수 있다. <span style="color:rgb(125, 125, 125)">(UE는 Nav Mesh Bounds Volume 액터가 레벨에 추가되는 즉시 Nav Mesh를 생성한다.)</span>
![[Pasted image 20250528134744.png|200]]
![[Pasted image 20250528134837.png|400]]

키보드 P를 눌러 Nav Mesh를 시각화 할 수 있다.
![[Pasted image 20250528135145.png|500]]

---
### Navigation Modifier Volumes
Nav Mesh에 Navigation Area Class를 적용한다.
이를 통해서 Nav Mesh 특정 영역의 비용을 다르게 설정할 수 있다.

설정 방법은 Place Actors 패널에서 Nav Modifier Volume을 찾아 레벨로 드래그 하면 영역을 지정할 수 있다. 
![[Pasted image 20250528145444.png|500]]

Details > Area Class
Area Class는 4개를 제공한다.
![[Pasted image 20250528145705.png|400]]

| AreaClass  |  explain |
|---|---|
|NavArea_Default|Nav Mesh와 동일한 비용 할당|
|NavArea_LowHeight|낮은 높이가 있는 영역(해당 볼륨 내에 내비게이션 데이터를 생성하지 않는다.)|
|NavArea_Null|볼륨 내의 빈 영역(내비게이션 데이터를 생성하지 않는다. 비용이 무한대로 적용)|
|NavArea_Obstacle|볼륨 내 높은 내비게이션 비용 할당|

---
### Navigation Link Proxy
Smart Link를 사용하여 Nav가 끊긴 영역을 이어줄 수 있다.
그럼 영역이 끊겨도 Lick가 있는 곳은 이동할 수 있다.

Place Actors > Nav Lick Proxy를 찾아 레벨로 드래그 하면 영역을 지정할 수 있다. 
드래그를 해보면 `PointLinks.Left/Right` 가 생긴것을 볼 수 있다. 
기본적으로 2개가 생성된다.
![[Pasted image 20250528151728.png|400]]

영역을 더 넓히고 싶다면 Details > Point Links 에서 Index를 추가할 수 있다. 또한 각각의 Pint Link 세부 설정을 할 수 있다. 
![[Pasted image 20250528152117.png|400]]
![[Pasted image 20250528152038.png|400]]

Direction에서는 Right와 Left를 통해 어느 방향으로 이동을 가능하게 할 지 설정할 수 있다. 아래는 Left to Right로 설정했을 때의 모양이다. 화살표가 Left에서 시작하여 Right로 향하는 것을 볼 수 있다.
![[Pasted image 20250528152438.png|400]]
![[Pasted image 20250528152646.png|400]]