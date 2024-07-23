
충돌체 Actor 생성

##### <mark style="background: #FFB86CA6;">Collison.h </mark>
###### 3, 6 :
충돌체 Actor를 생성해 주기 위해서는 Collision와 Mesh를 추가해 주어야한다.(이는 blueprint에서 Add 해주는 것과 동일한 작업이다.)
이 액터에는 Sphere Collision와 Static Mesh를 추가해 주었다.

###### 9 : 
플레이어가 액터에 닿았을 때 파티션 실행할 수 있도록 Particle을 선언해준다.

###### 11 : 
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

##### <mark style="background: #FFB86CA6;">Collison.cpp</mark> 
###### 5-6 : Static Mesh
생성자에서 Collision에 대해 초기화를 시켜줄 것이다.
- #UStataicMeshComponent 를 사용해서 #StaticMesh 를 만들어 준다.
- #RootComponent 로 정의해준다.

###### 8-10 : Sphere Collision
생성자에서 Mesh에 대해 초기화를 시켜줄 것이다.
-  #USphereComponent 를 사용해서 Collision을 만들어준다.
- Sphere Collision 반지름을 100.0f로 설정해준다.
- 그런 뒤 #RootComponent 밑에 붙여준다. 그렇지 않으면 Mesh와 Collision이 독립적으로 움직여 원치않는 움직임을 발생시킬 수 있다.

###### 12 : overlap 함수 정의
Collision이 오버랩되었을 때 실행될 함수를 정의해준다.
해당 코드는 blueprint에서 _On Component Begin Overlap_ 와 같은 역할을 한다. 

###### 14-15 : 파티클 정의
- _ConstructorHelpers::FObjectFinder<>_ 는 특정 타입의 에셋을 로드하는데 사용되는 헬퍼 클래스이다.
- (TEXT(""))를 통해 파티션 에셋이 있는 경로를 지정해 준다.
- 파티클 에셋을 이전에 선언해둔 파티클 변수에 붙여준다.
- 
###### 14-20 : 
오버랩 되었을 때 실행되는 함수 부분으로 OtherActor가 ACollisionCharacter가 맞는지 확인하고, 맞다면 스크린에 DebugMessage를 띄우고 해당 액터를 파괴하는 명령을 추가해주었다.

```c++ title:'Collison.cpp'
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
