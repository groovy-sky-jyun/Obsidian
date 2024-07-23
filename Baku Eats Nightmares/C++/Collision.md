
충돌체 Actor 생성

##### Collison.h : 3, 6, 9
충돌체 Actor를 생성해 주기 위해서는 Collision와 Mesh를 추가해 주어야한다.(이는 blueprint에서 Add 해주는 것과 동일한 작업이다.)
이 액터에는 Sphere Collision와 Static Mesh를 추가해 주었다.
그런 다음 오버랩됐을 때 실행될 함수를 정의해준다.
```c++ hl:3,6,9 title:'Collison.h'
public: 
	UPROPERTY(VisibleAnywhere, BlueprintReadWrite)
	USphereComponent* CollisionSphere; //Class 유무?

	UPROPERTY(VisibleAnywhrer, BlueprintReadWrite)
	UStataicMeshComponent* mStaticMesh;

	UFUNCTION()
		void OnOverlapBegin(UPrimitiveComponent* OverlappedComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult);
```

<br>

##### Collison.cpp : 
생성자에서 Collision와 Mesh에 대해 초기화를 시켜줄 것이다.
우선은 #UStataicMeshComponent 를 사용해서 StaticMesh를 만들어 준 뒤 #RootComponent 로 만들어준다.

그런 다음 #USphereComponent 를 사용해서 Collision을 만들어준다.
Sphere Collision 반지름을 100.0f로 설정해준다.
그런 뒤 #RootComponent 밑에 붙여준다.
#RootComponent 밑에 붙여주지 않으면 Mesh와 Collision이 독립적으로 움직여 원치않는 움직임을 발생시킬 수 있다.

Collision이 오버랩되었을 때 실행될 함수를 정의해준다.
해당 코드는 blueprint에서 On Component Begin Overlap와 같은 역할을 한다. 

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
	if(OtherActor->IsA(ACollisionCharacter::StaticClass())){
		GEngine->AddOnScreenDebugMessage(-1, 5.f, FColor::Red, TEXT("Collision Touch!!"))
		Destroy();
	}
}
```
