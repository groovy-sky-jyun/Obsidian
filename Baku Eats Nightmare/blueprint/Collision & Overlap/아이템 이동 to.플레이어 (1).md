
<br>

#### 기능 설명
플레이어가 길을 다니면서 맵에 배치된 아이템에 가까이 다가가면 아이템을 먹는다.
1. 플레이어가 아이템과 닿으면 아이템이 사라지도록 한다.
2. 플레이어가 아이템과 닿으면 아이템이 플레이어에게 이끌리듯이 위치가 이동되도록 한다.(자성효과)


#### Actor(아이템) 종류
[[Mesh와 Collision]]
>### #StaticMesh 
>- 정적인 object (움직이지 않는 물체)
>- 충돌이 감지되면 서로 튕겨진다.
>  ###### Why? 
>  Details>Collision>Collision Presets의 default값이 '<mark style="background: #ABF7F7A6;">OverlayAllDynamic</mark>'이기 때문이다.

>### #SkeletalMesh
>- 동적인 object(움직임이 있는 물체)
>- 충돌이 감지되어도 서로 튕겨지지 않는다.
>  ###### Why?
> Details>Collision>Collision Presets의 default값이 '<mark style="background: #ABF7F7A6;">NoCollision</mark>'이기 때문이다.

<br>

### 1. #Destroy Actor
좌측의 Add 버튼을 통해 collision을 추가해 준다.
![[Pasted image 20240704040431.png|300]]

<br>

이제 mesh가 아닌 colllision을 선택하여 collision의 우측의 Details 바에서 `events > On Component Begin Overlap > + `버튼을 선택한다. (mesh 선택 x)
해당 이벤트는 액터가 다른 무언가와 overlap이 되었을 때 발생하는 event를 정의할 수 있다.
![[Pasted image 20240704142830.png|300]]
>#### mesh가 아닌 collisoin을 사용하는 이유
플레이어와 물체(mesh)가 닿을 때 파괴되는 것이 아닌 어느정도 플레이어와 거리가 있어도 collision에 닿았을 때 파괴되도록 하기 위함이다.
충돌 감지는 대부분 collision을 사용한다.

<br>

그럼 Event Graph에 사진과 같은 빨간색 노드가 생긴다. 
여기에서 해당 액터와 충돌이 발생하는 `Other Actor`는 플레이어여야한다.
그러기 위해서 `Get Player Character` 노드를 추가하여 Other Actor가 플레이어 캐릭터라면 이라는 `==` 조건을 정의해준다.
![[Pasted image 20240704035436.png|300]]

<br>

_부딪힌 액터가 플레이어 라면_ 이라는 조건문을 사용할 것이기 때문에 `Branch` 를 사용해준다.  Branch 는 조건문의 결과값을 True/False로 나누어 기능을 실행할 수 있게 해준다. 
조건을 Branch 의 `Condition`와 연결해준다. 
- (참)이라면 `Destroy Actor`를 실행해준다. 
- 파괴되는것은 self 액터이기 때문에 Target은 따로 변경하지 않는다. 
- (거짓)이라면 아무런 일도 일어나지 않기 때문에 Branch의 False 또한 따로 지정해주지 않는다. 
- ![[Pasted image 20240704035911.png]]

<br>

액터가 플레이어와 부딪혔을 때 본 액터가 파괴되도록 하는 전체 blueprint는 다음과 같다.
![[Pasted image 20240704141327.png]]