<br>

#### 기능 설명
여러개의 오브젝트들이 Level을 시작할 때 마다 랜덤한 Static Mesh 이미지를 가지도록 구현해 볼 것이다. 이는 배열을 만들고 여러개의 Static Mesh 이미지를 담아둔 다음 그 중에서 랜덤으로 선택되도록 할 것이다.

이러한 기능은 여러개의 오브젝트들이 반복적으로 나타나는 맵에서 랜덤으로 이미지를 띄우고 싶을 때 사용할 수 있다.
1. 여러개의 액터 생성 후 랜덤으로 이미지가 선택되도록 설정

### 1. #Random #StaticMesh 

#### (1) 여러개의 액터 생성
랜덤 이미지를 받을 액터를 4개정도 만들어준다. (이미지는 나중에 랜덤으로 생성되게 할 것이기때문에 없어도 되지만 편의상 눈에 보이게 하기위해 임의의 Static Mesh를 사용하였다.)
![[Pasted image 20240721180848.png]]

<br>

#### (2) Random 함수 생성
이번에는 event graph에 바로 작성하지 않고 함수를 따로 만들어주도록 하겠다. FUNCTIONS에서 함수를 추가해준다. (이름은 마음대로)

![[Pasted image 20240721192507.png|400]]

<br>

#### (3) ARRAY VARIABLES
좌측의 Static Mesh를 끌어오면 변수로 사용할 수 있다. `Set Static Mesh` 노드를 통해서 Mesh를 지정해 줄 수 있다.
![[Pasted image 20240721192824.png]]

<br>

그런데 우리는 랜덤으로 Mesh가 선택되기를 원한다. 그럼 우선 랜덤으로 나올 Mesh 배열을 만들어 주어야 한다. 좌측의 VARIABLES에서 추가 버튼을 통해 Reference를 추가해 줄것이다. 이름은 마음대로 짓고 형태를 Static Mesh의 `Object Reference`로 지정해준다.
![[Pasted image 20240721193004.png|500]]

<br>

그럼 오른쪽의 `Variable Type`에서 `Array`로 변경해줄 수 있다. 
![[Pasted image 20240721193222.png|400]]

<br>

이렇게만 하면 Array를 채울 수 있는 버튼이 생기지 않는다.
 좌측상단의 `Compile`을 눌러주면 `Default Value` 에서 Mesh를 추가할 수 있는 버튼이 생겨난 것을 확인할 수 있다. 여기에서 랜덤으로 나왔으면 하는 Mesh들을 추가해주도록 하자.
![[Pasted image 20240721193415.png|400]]
![[Pasted image 20240721193608.png||400]]

<br>

#### (4) Random Array
이제 Array가 완성되었으니 여기에서 Random값을 가져와 보도록 하자. 좌측의 VARIABLES에서 array를 끌어와주고 (Get Random Array),  `Random Array Item`을 Set Static Mesh의 `New Mesh`와 연결시켜준다. 그럼 Set Static Mesh의 `New Mesh`는 array에서 random값을 입력받게된다.
![[Pasted image 20240721193905.png|500]]
![[Pasted image 20240721193946.png|500]]

<br>

나머지 Static Mesh들도 `Set Static Mesh`를 지정해주려 하는데 위에 만들어진것에다가 다른 Static Mesh들을 연결하면 이들은 랜덤이지만 다 같은 Mesh를 입력받게된다. 그래서 따로 만들어주어야하는데 여러개가 동시다발적으로 시작되어야 하므로 `Sequence` 노드를 사용해 줄것이다. `Add pin`을 통하여 총 4개의 pin과 `Set Static Mesh`를 만들어준다.(복사 붙여넣기)
![[Pasted image 20240721195024.png|200]]
![[Pasted image 20240721195142.png|400]]

<br>

Static Mesh Reference를 쉽게 변경하려면 좌측의 Static Mesh를 끌어와 아래 사진과 같이 앞쪽 머리 부분에 가져다 주면 쉽게 변경할 수 있다.
![[Pasted image 20240721194647.png|300]]

<br>

#### (5) 전체 blueprint
`Sequence`와 4개의 `Set Static Mesh` 노드를 이어 붙여주면 최종 `RandomSpawn`함수는 다음과 같다. 
![[Pasted image 20240721195408.png]]

<br>

여기서 중요한점은 우리는 지금 함수를 만들어 주었기 때문에 이 함수를 실제 event graph에 적용시켜주어야 한다는 점이다.
좌측의 event graph를 선택한 뒤 FUNCTIONS에 있는 `RandomSpawn`을 끌어와 연결시켜주면 끝이난다.
![[Pasted image 20240721195636.png||500]]