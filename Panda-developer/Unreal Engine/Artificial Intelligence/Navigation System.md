Agent가 경로 찾기를 통해 이동할 수 있도록 범위를 지정해준다.
그럼 범위 내에서 시작 위치와 목적지 사이 경로를 찾는다.

### #NavMesh
Navigation Mesh를 줄여서 Nav Mesh라고 한다.
World Collision Geometry에서 생성되며, PolygonMesh를 통해 레벨에서 이동 가능한 공간을 나타낸다. <span style="color:rgb(125, 125, 125)">(Nav Mesh를 타일로 분할할 수 있다.)</span>

### 비용 Cost
Nav Mesh를 사용하여 한 지점에서 다른 지점으로 이동할 때, 지나가는 경로상의 모든 영역 비용의 합을 뜻한다. 

<span style="color:rgb(188, 149, 218)">Agent는 항상 비용이 가장 낮은 경로를 찾아 이동</span>하기 때문에 영역의 비용을 조절하여 동선을 수정할 수 있다.

예를 들어 거친 지형에는 AI가 오지 못하도록 영역 비용을 높여버리면 AI가 찾은 비용이 가장 낮은 경로에 거친 지형이 포함될 확률이 낮아지므로 해당 영역을 피할 수 있다.

그러나 거친 지형을 포함하는것이 경로 상 비용이 제일 낮다면 비용이 높은 영역이라 하더라도 지날 수 있다.

만약 비용이 가장 낮은 동선이 여러개 있다면 직선경로를 우선시 한다.

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
특정 영역의 비용을 다르게 주고자 할 때, 비용을 다르게 줄 영역을 표시하는 역할을 한다. 해당 영역은 AreaClass 를 통해서 비용을 커스텀할 수 있다.<span style="color:rgb(125, 125, 125)"> (영역 표시는 Modifier Volumes, 비용 설정은 Modifier Volumes 내의 Area Class 변수로 설정)</span>

Place Actors 패널에서 Nav Modifier Volume을 찾아 레벨로 드래그 하면 영역을 지정할 수 있다. 
![[Pasted image 20250528145444.png|400]]

Nav Modifier Volume의 비용은 #AreaClass 로 설정할 수 있는데 기본적으로 4개의 Class가 제공된다. 이 외의 다른 AreaClass 를 사용하고 싶다면 Blueprint로 생성하여 적용할 수 있다.
 ![[Pasted image 20250528145705.png|400]]
 



---

### #AreaClass
Blueprint(Nav Area)를 통해 Area Class를 생성할 수 있다. Area Class는 Class마다 Cost를 다르게 설정할 수 있고, 개발 환경에서 색상별로 영역을 구분할 수 있도록 색상을 지정할 수 있다.
![[Pasted image 20250529125511.png|400]]
![[Pasted image 20250529125643.png|400]]

예를 들어 BP_Area_Land1, BP_Area_Land2, BP_Area_Nutral을 생성했다면 Navigation Modifier Volumes의 Area Class에서 다음과 같이 생성한 Area Class도 사용할 수 있다.
이런식으로 영역별 비용 커스텀이 가능한 것이다.
![[Pasted image 20250529130017.png|400]]
![[Pasted image 20250529130304.png|400]]

> ##### 기본 제공 Area Class
> Area Class가 기본적으로 제공하는 4개의 클래스는 다음과 같다.
>
| AreaClass  |  explain |
|---|---|
|NavArea_Default|Nav Mesh와 동일한 비용 할당|
|NavArea_LowHeight|낮은 높이가 있는 영역(해당 볼륨 내에 내비게이션 데이터를 생성하지 않는다.)|
|NavArea_Null|볼륨 내의 빈 영역(내비게이션 데이터를 생성하지 않는다. 비용이 무한대로 적용)|
|NavArea_Obstacle|볼륨 내 높은 내비게이션 비용 할당|

---
### Navigation Query Filter
Agent 마다 Area Class에 대한 비용 값을 Override 하여 다르게 적용시킬 수 있다.
즉, Agent마다 레벨에서 이동하는 경로를 다르게 커스터마이징 할 수  있다.

예를 들어, Agent1 에게는 E구역의 영역비용이 10 이지만 Agent2 에게는 E구역의 영역비용을 20으로 줄 수 있는 것이다.

Query Filter는 Blueprint에서 Navigation Query Filter를 사용하여 생성할 수 있다.
 ![[Pasted image 20250529131106.png|400]]

생성한 BP를 더블 클릭하면 다음과 같은 Details를 볼 수 있다.
플러스 버튼을 눌러 커스터마이징할 Area Class를 추가할 수 있다.
`Travel Cost Override`는 AreaClass의 Cost를 Override하겠다는 뜻이다.
`Entering Cost Override`는 영역에 들어갈때 한번 발생하는 Cost를 Override하겠다는 뜻이다. <span style="color:rgb(125, 125, 125)">(false는 Override x)</span>
![[Pasted image 20250529145642.png|400]]

> ##### Agent 적용 예시
이는 Agent에게 적용시킬 Query Filter 값으로 Agent_1에게 BP_Area_Nutral, BP_Area_Lane2의 Cost를 100으로 Override하고 싶다면 다음과 같이 설정한다.
![[Pasted image 20250529144329.png|600]]
>
Agent_2에게는 다르게 적용하고 싶다면 새로운 QueryFilter를 생성해서 커스텀 해준다.
![[Pasted image 20250529145015.png|600]]
>
>이를 Agent에게 적용시키기 위해서는 Agent 클래스에 `Navigation Query Filter`타입의 변수가 있어야 한다. 이 변수를 이용하여 `Move to Actor` 함수의 매개변수로 넘겨주면 해당 규칙이 적용된다.
>![[Pasted image 20250529145258.png|400]]
>![[Pasted image 20250529145359.png|400]]

---
### Navigation Link Proxy
Smart Link를 사용하여 Nav가 끊긴 영역을 이어줄 수 있다.
그럼 영역이 끊겨도 Lick가 있는 곳은 이동할 수 있다.

Place Actors > Nav Lick Proxy를 찾아 레벨로 드래그 하면 영역을 지정할 수 있다. 
드래그를 해보면 `PointLinks[0].Left/Right` 가 생긴것을 볼 수 있다. 
![[Pasted image 20250528151728.png|400]]

Link를 추가하고 싶다면 Details > Point Links 에서 Index를 추가할 수 있다. 또한 각각의 Pint Link 세부 설정을 할 수 있다. 
![[Pasted image 20250528152117.png|400]]
![[Pasted image 20250528152038.png|400]]

Direction에서는 Right와 Left를 통해 어느 방향으로 이동을 가능하게 할 지 설정할 수 있다. 아래는 Left to Right로 설정했을 때의 모양이다. 화살표가 Left에서 시작하여 Right로 향하는 것을 볼 수 있다.
![[Pasted image 20250528152438.png|400]]
![[Pasted image 20250528152646.png|400]]