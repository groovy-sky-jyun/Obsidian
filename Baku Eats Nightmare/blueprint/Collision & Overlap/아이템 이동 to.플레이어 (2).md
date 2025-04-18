### 2. #Animation Actor
#### (1) TimeLine
액터가 사용자에서 자성처럼 이끌리듯 움직이는 표현을 구현하기 위해 `Timeline`을 사용할 것이다. 
Timeline이란 간단한 시간 기반 애니메이션을 빠르게 디자인 할 수 있는 노드이다.
추가한 Timeline 노드를 더블 클릭하면 Timeline 에디터 창이 생성된다.
![[Pasted image 20240704143303.png|350]]
![[Pasted image 20240704143344.png|300]]
![[Pasted image 20240704143402.png]]

<br>

우리는 특정시간동안 물체가 플레이어에게 이동하는 크기(속력, 각도)만 지정해 주면 되므로 Timeline 에디터 창에서 `+Track > Add Float Track`을 추가해준다.
>#### +Track 종류
><u>Float Track</u>
>: 시간에 따라 변하는 간단한 변수를 생성할 때 사용(시간과 변화하는 값의 크기 지정, vector와 다르게 전체적인 값 하나만 변경이 필요할 때 사용)
>
><u>Vector Track</u>
>: 액터의 애니메이션으로 x, y, z 값 변경이 필요할 때 사용한다.

![[Pasted image 20240704143436.png|500]]

<br>

Timeline 에디터 창에서 `shift를 누르고 마우스 좌클릭`을 하면 새로운 key를 마우스 포인터 위치에 생성할 수 있다.
우리는 액터가 _(1)원래 위치_ 에 있다가 _(2)플레이어에게 다가가는_ 움직임만 필요로 하기 때문에 key는 두개가 필요하다.
![[Pasted image 20240704143648.png]]
- 첫번째 키는 원래위치에 있어야 하므로 Time와 Vlue 전부 0으로 지정해준다.
	- ![[Pasted image 20240704143719.png|300]]
- 두번째 키는 Time와 Value를 각각 0.15, 0.5로 지정해준다.
	- ![[Pasted image 20240704144017.png]]
- 전체적인 Length의 값은 0.15로 지정해준다.
- 이러한 값들은 절대적인 것이 아니고 상황에 맞게 조절하면 된다. (현재 이 프로젝트에서 아이템이 사용자에게 움직이는 값은 해당값이 적당하다고 판단한 것이다.)
- >#### Length
>해당 애니메이션이 실행되는 총 시간을 의미한다. 
>해당 길이만큼의 시간이 끝나면 다음 노드로 동작이 넘어간다.
><br>
>#### Time
>이전 key에서 본 key로 value만큼 실행되는데 걸리는 시간을 정의할 수 있다. 
>value만큼 천천히 움직이게 하고 싶다면 time을 길게하고, 빠르게 움직이게 하고 싶다면 time을 빠르게 하면된다. 
>즉, __`속도`__ 를 생각하면 된다.
><br>
>#### Value
>액터가 움직이는 크기를 정의할 수 있다. 
>즉, __`각도`__ 를 생각하면 된다. 
>값이 작을 수록 각도가 0에 가까워진다.
><br>
>#### Length와 Time의 관계
>두개의 개념이 비슷하다고 생각이 들어 헷갈릴 수 있다.
>Length는 애니메이션의 속도에는 영향을 주지 않는다. 
>- 만약 `Length<Time`이라면 액터가 vector 방향으로 움직이고 있다가 목적지까지 도달하지 못했는다 Length가 다되어서 동작이 마저 끝나지않고 도중에 다음 노드 동작으로 넘어가게 된다.
>- 만약 `Length>Time`이라면 액터가 vector 방향으로 다 움직여서 동작이 끝났지만 Length가 남아서 다음 노드 동작으로  넘어가지않고 가만히 있다가 Length 시간이 다되어야 다음 노드 동작으로 넘어가게 된다.

Timeline을 통해 애니메이션을 지정해 주었으므로 이제 이 애니메이션을 액터의 위치 변화와 연결시켜줄 것이다. 
Timeline의 `Update`는 `Set Actor Location`노드와 연결해주고, `Finished`는 `Destroy Actor`와 연결해준다.
![[Pasted image 20240704143128.png]]

<br>

#### (2) Lerp
이대로만 실행하면 액터의 애니메이션이 부자연스럽다. 
이를 좀 더 부드럽게 움직이게 해주기 위해 `Lerp(선형보간법)`를 추가해 줄 것이다.
Lerp의 `A`를 밖으로 끌어내면 A값을 지정해 줄 수 있는 검색창이 나온다.
액터는 자기 위치에서 플레이어 위치로 이동해야하기 때문에 A에는 `Get Actor Start Location`을 변수값으로 준다.
![[Pasted image 20240704144305.png|400]]
`B`는 플레이어의 위치를 가져와야 하므로 `Get Actor Location`를 사용해준다. 여기에서 `Target`을 _1. Destroy Actor_ 에서 사용했던 _Get Player Charactor_ 와 연결해줄 것이다.
![[Pasted image 20240704144435.png|400]]
![[Pasted image 20240704144958.png]]

그리고 나서 `Lerp의 Alpha`와 `Timeline에서 생성한 Track`을 연결시켜준다.
아래 사진에서 Track의 이름이 Actor Move인 이유는 Track을 생성할때 이름을 저렇게 지정해 줬기 때문이다.
그리고 `Lerp의 Return Value`를 `Set Actor Location의 New Location`와 연결해준다. 
Timeline을 통해서 애니메이션을 지정해주고 Lerp를 통해서 위치를 지정해줬다고 생각하면 된다.
![[Pasted image 20240704144410.png|500]]

여기서 끝인줄 알았겠지만 이렇게만 실행하면 액터의 움직임에 변화가 없다. 그 이유는 `Actor Start Location`변수 값을 초기화 시켜주지 않았기 때문이다.
왼쪽 목록에서 Actor Start Location를 드래그 하면 아래 사진과 같이 Get/Set Actor Start Location 메뉴창이 나온다. 우리는 초기화 시켜주어야 하므로 `Set Actor Start Location`를 선택해 준다.
![[Pasted image 20240704144601.png|500]]

Set에서 Lerp Start Location값은 액터 본인의 원래 기존 위치이므로 Get Actor Location(self)를 추가하여 연결시켜준다.
그리고 set은 branch에서 True가 되었을 때 정의해주고, 정의가 끝나면 Timeline을 실행해주면 되므로 아래의 사진과 같이 연결해준다.
![[Pasted image 20240704144902.png|500]]


<br>

#### (3) 전체 blueprint
플레이어가 액터와 닿았을 때 자성처럼 액터가 플레이어에게 끌려가다가 사라지는 기능을 구현하는 blueprint의 전체 event graph는 다음과 같다.
![[Pasted image 20240704145122.png]]