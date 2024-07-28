
### 기능 설명
플레이어와 닿으면 폭발하며 사라지는 Actor 구현

#### <mark style="background: #FFB86CA6;">[Collison.h] </mark>
###### <mark style="background: #FFB86CA6;">3, 6</mark> : 충돌체 Actor 생성
Actor에 Collision와 Mesh를 추가해 주어야한다.(이는 blueprint에서 Add 해주는 것과 동일한 작업이다.) 이 액터에는 #SphereCollision 와 #StaticMesh 를 추가해 주었다.

###### <mark style="background: #FFB86CA6;">9</mark> : Particle 선언
플레이어가 액터에 닿았을 때 파티션 실행할 수 있도록 Particle을 선언해준다.

###### <mark style="background: #FFB86CA6;">11</mark> : Overlap 함수 선언언
오버랩됐을 때 실행될 함수를 정의해준다.

```c++ hl:3,6,9,11 title:'Collison.h'
public: 
	UPROPERTY(VisibleAnywhere, BlueprintReadWrite)
	USphereComponent* CollisionSphere; //Class 유무?

	UPROPERTY(VisibleAnywhrer, BlueprintReadWrite)
	UStataicMeshComponent* mStaticMesh;

	UPROPERTY(EditDefaultsOnly, Category="MyItem")
		UParticleSystem* ParticleFX;
		
	UFUNCTION()
		void OnOverlapBegin(UPrimitiveComponent* OverlappedComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult);
```

<br>

#### <mark style="background: #FF5582A6;">[Collison.cpp]</mark>
###### <mark style="background: #FF5582A6;">5-6</mark> : Static Mesh 초기화
생성자에서 Collision에 대해 초기화를 시켜줄 것이다.
- #UStataicMeshComponent 를 사용해서 #StaticMesh 를 만들어 준다.
- #RootComponent 로 정의해준다.

###### <mark style="background: #FF5582A6;"> 8-10</mark> : Sphere Collision 초기화
생성자에서 Mesh에 대해 초기화를 시켜줄 것이다.
-  #USphereComponent 를 사용해서 Collision을 만들어준다.
- Sphere Collision 반지름을 100.0f로 설정해준다.
- 그런 뒤 #RootComponent 밑에 붙여준다. 그렇지 않으면 Mesh와 Collision이 독립적으로 움직여 원치않는 움직임을 발생시킬 수 있다.

###### <mark style="background: #FF5582A6;">12</mark> : overlap 함수 정의
Collision이 오버랩되었을 때 실행될 함수를 정의해준다.
해당 코드는 blueprint에서 _On Component Begin Overlap_ 와 같은 역할을 한다. 

###### <mark style="background: #FF5582A6;">14-15</mark> : 파티클 정의
- _ConstructorHelpers::FObjectFinder<>_ 는 특정 타입의 _에셋을 로드_ 하는데 사용되는 헬퍼 클래스이다.
- (TEXT(""))를 통해 파티션 에셋이 있는 경로를 지정해 준다.
- 파티클 에셋을 이전에 선언해둔 파티클 변수에 붙여준다.
- 
###### <mark style="background: #FF5582A6;">18-26</mark> : overlap 함수 구현
오버랩 되었을 때 실행되는 함수 부분이다.
-  OtherActor가 ACollisionCharacter가 맞는지 확인
- 맞다면 ScreenMessage로 "Collision Touch!!"가 출력, 파티클이 실행, 액터 파괴
- >###### <mark style="background: #FF5582A6;">23</mark> : 파티클 스폰
>  파티클을 _GetActorLocation()_ 위치에 스폰한다는 명령을 뜻한다. 
>  (액터에 파티클을 붙인게 아니라 파괴될때 다른 클래스 파티클을 해당 액터 위치 실행시키는 것이다.)

```c++ hl:23 title:'Collison.cpp'
ACollisionActor::ACollosioinActor()//생성자 
{
	PrimaryActorTick.bCanEverTick = true; //기존에 있는 값

	mStaticMesh = CreateDefaultSubobject<UStataicMeshComponent>(TEXT("Obj"));
	RootComponent = mStaticMesh;

	CollisionSphere = CreateDefaultSubobject<USphereComponent>(TEXT("CollisionSphere"));
	CollisionSphere->InitSphereRadius(100.0f);
	CollisionSphere->SetupAttachment(RootComponent);

	CollisionSphere->OnComponentBeginOverlap.AddDynamic(this, &ACollisionActor::OnOverlapBegin);

	static ConstructorHelpers::FObjectFinder<UParticleSystem>ParticleAsset(TEXT(""));
	particleFX = ParticleAsset.Object;
}

void ACollisionActor::OnOverlapBegin(UPrimitiveComponent* OverlappedComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
	if(OtherActor->IsA(ACollisionCharacter::StaticClass())){
		GEngine->AddOnScreenDebugMessage(-1, 5.f, FColor::Red, TEXT("Collision Touch!!"))

		UGameplayStatics::SpawnEmitterAtLocation(GetWorld(), ParticleFX, GetActorLocation());
		Destroy();
	}
}
```

<br>

### + 추가 구현
지금은 새로운 파티클을 현재 액터 위치에 spawn 하는 것을 구현했다.
다른 방식으로 아래의 조건을 만족하도록 구현해보아라.
- 액터에 파티클을 붙이기
- 초기상태에는 파티클을 비활성화
-  overlap되었을때 활성화
```
ParticleSystem->SetupAttachment(RootComponent); 
ParticleSystem->bAutoActivate = false; // Disable auto-activation
```