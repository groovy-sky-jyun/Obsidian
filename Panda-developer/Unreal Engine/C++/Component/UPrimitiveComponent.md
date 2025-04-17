### #UPrimitiveComponent
#USceneComponent 의 자손으로 Geometry 표현이 있는 씬 컴포넌트이다. 

다음과 같은 함수를 제공한다.
- __비주얼 요소 렌더링__
- __물리적 오브젝트와의 충돌__
- __오버랩__ 

주로 다음과 같은 클래스에서 사용한다.
- [Collision] Box, Capsule, Sphere,
- [Mesh] Static, Skeletal

이러한 클래스들은 UPrimitiveComponent를 상속받고 있어서 UPrimitiveComponent에서 제공하는 함수를 사용해서 렌더링, 충돌, 오버랩 등의 기능을 사용할 수 있다.

>|      | Collision Component | Mesh Component |
|:-----:|:-----:|:-----:|
| 종류 | #BoxComponent, #CapsuleComponent, #SphereComponent | #StaticMeshComponent, #SkeletalMeshComponent |
| 렌더링 | 보이지 않는 Geometry 생성 (렌더링X) | 렌더링 가능한 사전 제작된 Geometry (.fbx 3D 모델 파일 등) |
| 목적 | 충돌 감지 전용 | 시각적 표현 |

---

### UPrimitiveComponent의 #Collision 와 연관된 함수
##### `SetCollisionEnabled(ECollisionEnabled::Type)`
- 충돌 처리 방식 설정
>###### #ECollisionEnabled
> |      |   Overlap/Trace   |   물리적 충돌   |
|:-----:|:-----:|:-----:|
|   NoCollision   |   비활성화   |   비활성화   |
|   QueryOnly   |   활성화   |   비활성화   |
|   PhysicsOnly   |   비활성화   |   활성화   |
|   QueryAndPhysics   |   활성화   |   활성화   |


##### `SetCollisionObjectType(ECollisionChannel)`
- 해당 컴포넌트의 충돌 객체 타입 설정
>###### #ECollisionChannel
> |      |   객체 종류  | 
|:-----:|:-----:|
|   ECC_Pawn   |   플레이어나 AI   | 
|   ECC_WorldStatic   |   벽, 지형 등 정적 오브젝트  |
|   ECC_WorldDynamic   |   움직이는 동적 오브젝트   | 
>이 외에는 


##### `SetCollisionResponseToAllchannels(ECollisionResponse)`
- 모든 채널에 대한 충돌 설정
>###### #ECollisionResponse
> |      |   충돌   | 오버랩| 
|:-----:|:-----:|:-----:|
| ECR_Block | 물리적 반응 발생 | 오버랩 이벤트 발생 x |
| ECR_Overlap | 물리적 반응 x | 오버랩 이벤트 발생 (겹쳐져서 플레이어가 통과됨) |
| ECR_Ignore |  무시  | 무시 | 

##### `SetCollisionResponseToChannel(ECC, ECR)`
- 특정 채널에 대한 충돌 설정

##### `SetSimulatePhysics(bool)`
- 물리 시뮬레이션 활성화/비활성화

##### `CanCharacterStepUpOn`
- 캐릭터가 이 컴포넌트 위로 밟고 올라갈수 있는지 여부 설정

##### `SetCollisionProfileName(FName, Bool)`
- 해당 컴포넌트의 충돌 설정을 지정된 Preset으로 적용
- SetCollisionEnabled, SetCollisionResponseToChannel 등을 개별로 설정하지 않고, 미리 정의된 설정을 사용
- CollisionEnabled, CollisionObjectType, CollisionResponse를 한번에 설정할 수 있다.
>###### Collision Profile Name
> |   Profile Name   |   의미   | 
|:-----:|:-----:|
| BlockAll | 모든 채널 Block|
| BlockAllDynamic | 동적 오브젝트만 Block | 
| NoCollision |  충돌 x  |  
| Pawn |  Pawn 처럼 설정 |  
| PhysicsActor |  물리적 액터처럼 설정 | 
| Vehicle |  탈것 처럼 설정  | 
| Custom |  Collision Profile Name을 설정하지 않았을 때 기본 값 | 
> <br>
> 이 프로필들은 하나마다 세세한 설정들이 다르게 되어있다.
> 자세한 설정을 알고싶다면 언리얼 엔진의 블루프린트에서 해당 컴포넌트의 Details 패널 > Collision Presets 에서 확인할 수 있다.
> 
> 예를 들어, BlockAll 의 경우 아래 값처럼 설정이 되어 있다.
> 
>![[Pasted image 20250417192410.png|400]]

---

### UPrimitiveComponent의 #Overlap 와 연관된 함수
##### `bGenerateOverlapEvents`
- 값이 true 면 BegineOverlap/EndOverlap 이벤트 발생

##### `SetGenerateOverlapEvents(bool)`
- bGenerateOverlapEvents 값 설정

##### `OnComponentBeginOverlap()`
- 델리게이트
- 다른 객체가 이 컴포넌트와 오버랩 됐을 때 실행되는 함수
- > 함수 실행 조건
> - bGenerateOverlapEvents = true
> - 충돌 채널 설정 = Overlap 활성화

##### `OnComponentEndOverlap()`
- 델리게이트
- 다른 객체가 이 컴포넌트와의 오버랩이 끝났을 때 실행되는 함수

##### `GetOverlappingActor()`
- 이 컴포넌트와 오버랩 중인 Actor 목록 배열로 반환

##### `GetOverlappingComponent()`
- 이 컴포넌트와 오버랩 중인 Component 목록 배열로 반환

##### `IsOverlappingActor(AActor*)`
- 특정 액터와 겹치고 있는지 확인

##### `IsOverlappingComponent(UPrimitiveComponent*)`
- 특정 컴포넌트와 겹치고 있는지 확인

