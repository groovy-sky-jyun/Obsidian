<br>

#### 기능 설명
Level을 플레이할 때 맵 어디에서든 특정 버튼을 누르면 원하는 기능이 실행되도록 할 것이다. 여기에서는 1번을 누르면 오브젝트가 생성되고 2번을 누르면 오브젝트가 Destroy되도록 구현할 것이다. 
1. 1번 누르면 오브젝트 생성 & 2번 누르면 오브젝트 파괴
2. 2번 누르고 나서 특정 시간만큼 지나면 파괴

### 1. #KeyInput #Create #Destroy
1번을 눌렀을 때 오브젝트가 생성되도록한다. 이때 생성되는 오브젝트는 이전에 만들어뒀던 세이브 포인터를 사용한다. 

참고 | [[세이브 포인터(1)_빙글빙글]]

#### (1) Note & Level Blueprint
1번을 누르면 세이브포인터가 맵상의 특정 위치에서 생성되야하므로 위치의 정보만 가지고 있는 note를 맵에 배치해준다. 
> #### Note
> 위치 정보만 가지고 있다. 그 외의 다른 정보는 가지고 있지 않는다.

![[Pasted image 20240721121903.png]]

<br>

Level 어디에서나 특정 key를 누르면 해당 기능이 실행되어야 하므로 이번에는 Level의 Blueprint를 수정해줄것이다.
![[Pasted image 20240721121653.png]]

<br>

#### (2) Key Input
`Keyboard Events`를 사용할 것이다. 
![[Pasted image 20240721122249.png]]

<br>

#### (3) Spawn
1번 키를 누르면 액터가 스폰되어야 하므로 `SpawnActor from Class`를 추가해준다. 그런 뒤 스폰될 액터 클래스를 `Class`에 지정해준다. 우리는 이전에 만들었던 BP Wave Point 액터를 사용해 줄 것이다.
![[Pasted image 20240721122600.png|400]]
> ### <mark style="background: #ABF7F7A6;">SpawnActor from Class</mark>
> ##### Class
> 생성하고자 하는 액터 클래스
> ##### Spawn Transform
> 생성하고자 하는 액터의 위치
> ##### ReturnValue
> 생성된 객체 반

<br>

`Spawn Transform`은 현재 note의 위치에 생성되어야 한다. 그렇기 때문에 `Get Actor Transform`을 연결시켜주고 `Target`은 note여야 하므로 맵에 note가 선택되어 있는 상태에서 좌클릭을 하면 `Create a Reference to Note`라는 함수가 보인다. 이를 클릭하여 연결시켜준다. 
![[Pasted image 20240721123656.png]]
![[Pasted image 20240721123749.png]]

<br>

#### (4) VARIABLES 저장
이렇게만하면 1번 키를 눌렀을 때 note 위치에 세이브 포인트 액터가 생성된다. 여기에서 한발짝 더 나아가 우리는 이 액터를 2번을 누르면 Destroy 시켜줄 것이다. 그러기 위해서는 이 _생성된 액터를 저장_ 하고 있어야 한다. _그래야 이 저장된 액터를 찾아서 Destroy_ 시켜주기 때문이다.

저장을 하기 위해서는 SpawnActor from Class의 `Return Value`에서 우클릭을 한다. 그런 뒤 `Promote to Variable`을 선택하여 변수로 승격시켜줄 수 있다. 그러면 좌측 VARIABLES에 새롭게 변수로 추가된 것을 확인할 수 있다.
![[Pasted image 20240721124127.png]]
![[Pasted image 20240721124257.png]]
![[Pasted image 20240721124432.png]]

<br>

#### (5) Destroy
이번에는 2번을 누르면 이 객체를 삭제하도록 하겠다. `2번 Keyboard Events`를 생성해주고, Pressed 되면 `Destroy Actor` 되도록 해준다. 이때 `Target`은 좌측에 VARIABLES에 추가된 변수를 연결시켜준다.
![[Pasted image 20240721124616.png||350]]
![[Pasted image 20240721124732.png|350]]

<br>

#### (6) 전체 blueprint
1번키를 눌렀을 때 오브젝트가 생성되고 2번키를 눌렀을 때 생성되었던 오브젝트가 파괴되는 event graph는 다음과 같다.
![[Pasted image 20240721124852.png]]
![[Pasted image 20240721124902.png]]
