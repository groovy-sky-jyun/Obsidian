
충돌체 작성

```c++ title:'Collison.h'
public: 
	UPROPERTY(VisibleAnywhere, BlueprintReadWrite)
	USphereComponent* CollisionSphere; //Class 유무?

	UPROPERTY(VisibleAnywhrer, BlueprintReadWrite)
	UStataicMeshComponent* mStaticMesh;

	UFUNCTION()
		void OnOverlapBegin(UPrimitiveComponent* OverlappedComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult);
```


```c++ title:'Collison.cpp'
ACollisionActor::ACollosioinActor() //생성 
	PrimaryActorTick.bCanEverTick = true; //기존에 있는 값

	mStaticMesh = CreateDefaultSubobject<UStataicMeshComponent>(TEXT("Obj"));
	RootComponent = mStaticMesh;

	CollisionSphere = CreateDefaultSubobject<USphereComponent>(TEXT("CollisionSphere"));
	CollisionSphere->InitSphereRadius(100.0f);
	CollisionSphere->SetupAttachment(RootComponent);

	CollisionSphere->OnComponentBeginOverlap.AddDynamic(this, &ACollisionActor::OnOverlapBegin);


void ACollisionActor::OnOverlapBegin(UPrimitiveComponent* OverlappedComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
	if(OtherActor->IsA(ACollision::StaticClass())){
		GEngine->AddOnScreenDebugMessage(-1, 5.f, f)
	}
}
```