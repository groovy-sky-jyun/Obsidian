<br>

## 기능 설명

###### 캐릭터의 기본 동작 INPUT  설정

기본으로 제공되는 Character 클래스를 상속받을 것이기 때문에 _<span style="color:rgb(193, 173, 240)">부모 클래스를 Character로 지정</span>_ 한 후 c++ 파일을 생성해준다.
<br>

### <span style="color:rgb(135, 75, 195)">[FPSProjectile.h]</span>
##### <span style="color:rgb(217, 152, 99)">19-34</span> : <mark style="background: #FFB86CA6;">캐릭터 Input 함수 정의</mark>

<br>

``` c++ title:FPSCharacter.h  hl:19-34
#pragma once   

#include "CoreMinimal.h"
#include "GameFramework/Character.h"
#include "Camera/CameraComponent.h"
#include "Components/CapsuleComponent.h"
#include "FPSProjectile.h"
#include "FPSCharacter.generated.h"

UCLASS()
class BAKUEATSNIGHTMARES_API AFPSCharacter : public ACharacter
{
   ...

public:   
   // Called every frame
   virtual void Tick(float DeltaTime) override;

   // Called to bind functionality to input
   virtual void SetupPlayerInputComponent(class UInputComponent* PlayerInputComponent) override;

   // 앞뒤 이동 입력 처리
   UFUNCTION()
   void MoveForward(float value);

   // 좌우 이동 입력 처리
   UFUNCTION()
   void MoveRight(float value);

   UFUNCTION()
   void StartJump();

   UFUNCTION()
   void StopJump(); 
   
   ...
```

### 캐릭터 INPUT 동작 
``` c title:FPSCharacter.h
public: 

   // Called to bind functionality to input
   virtual void SetupPlayerInputComponent(class UInputComponent* PlayerInputComponent) override;

   // 앞뒤 이동 입력 처리
   UFUNCTION()
   void MoveForward(float value);

   // 좌우 이동 입력 처리
   UFUNCTION()
   void MoveRight(float value);

   UFUNCTION()
   void StartJump();

   UFUNCTION()
   void StopJump();
```

``` c title:FPSCharacter.cpp
// Called to bind functionality to input
void AFPSCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
   Super::SetupPlayerInputComponent(PlayerInputComponent);
    
   // 'movement' 바인딩 구성
   PlayerInputComponent->BindAxis("MoveForward", this, &AFPSCharacter::MoveForward);
   PlayerInputComponent->BindAxis("MoveRight", this, &AFPSCharacter::MoveRight);

   // 'look' 바인딩 구성
   PlayerInputComponent->BindAxis("Turn", this, &AFPSCharacter::AddControllerYawInput);
   PlayerInputComponent->BindAxis("Look", this, &AFPSCharacter::AddControllerPitchInput);

   // 'action' 바인딩 구성
   PlayerInputComponent->BindAction("Jump", IE_Pressed, this, &AFPSCharacter::StartJump);
   PlayerInputComponent->BindAction("Jump", IE_Released, this, &AFPSCharacter::StopJump);
}
```
_SetupPlayerInputComponent(class UInputComponent* PlayerInputComponent)_ 함수는 언리얼 편집기에서 지정한 Input들과 함수를 연결시켜준다. 사용자의 Input으로는 키보드, 마우스, 콘솔 입력 등이 있다. 

예로 `PlayerInputComponent->BindAxis("MoveForward", this, &AFPSCharacter::MoveForward);` 라는 코드는 
1. 언리얼에서 "MoveForward" 라는 이름의 Input을 지정해주었으면 
2. 이를 this (현재 코드가 적힌 클래스를 사용하는 캐릭터) 에게 적용시키며 
3. 해당 키가 눌렸을 때 `AFPSCharacter 클래스`의 `MoveForward 함수`를 실행한다는 뜻이다.

이를 이용해 캐릭터 앞, 뒤 , 좌, 우 위치 이동을 설정해주고, 마우스의 움직임(캐릭터 시야) 위, 아래, 좌, 우를 설정해준다. BindAction으로는 Jump 움직임을 설정해준다.

> #### BindAxis 와 BindAction의 차이
> ##### <mark style="background: #ADCCFFA6;">BindAxis</mark>
> 플레이어가 특정 키를 입력하면 이벤트가 발생하여 키값을 바인딩하게된다.
> ##### <mark style="background: #ADCCFFA6;">BindAction</mark>
> BindAxis와 같은 기능을 제공하는데 차이점이 있다. 바로 키를 눌렀을 때와 땠을때의 동작을 다르게 설정할 수 있는 점이다. 이는 BindAction이 EInputEvent를 사용하기 때문에 가능한것이다. 
> EInput은 IE_Pressed와 IE_Released 두가지를 제공하고 있다.
> 이를 사용할 때에는 한가지 이름(언리얼 편집기에 지정된 Input 이름)에 대해 바인딩 함수를 2개(IE_Pressed, IE_Released)를 지정해주어야 한다.
>  -  IE_Pressed : 눌렀을 떄 동작 
>  -  IE_Released : 땠을 때 동작

##### <span style="color:rgb(217, 152, 99)">45-46</span> : <mark style="background: #FFB86CA6;">UObject </mark>
UCameraComponent 클래스의 포인터 변수를 선언한다.
>#### 왜 포인터로 선언하는가?
>#UObject 기반의 클래스의 경우 언리얼의 #GarbageCollectionSystem 과 관련이 있어 보통 포인터로 선언된다. 
>
>#GarbageCollectionSystem 은 포인터를 추적하여 더이상 참조되지 않는 객체를 자동으로 해제한다. 그러므로써 개발자가 직접 메모리를 할당하고 해제해야하는 부담을 줄여준다.  
>
>#UPROPERTY 매크로를 통해 #GarbageCollectionSystem 이 해당 객체를 추적할 수 있도록 해준다. 만약 #UObject 를 #UPROPERTY 로 선언하지 않으면  #GarbageCollectionSystem 이 해당 객체를 추적하지 않게 되어 메모리 누수 등의 문제가 발생할 수 있다.
>
```
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Character.h"
#include "Camera/CameraComponent.h"
#include "Components/CapsuleComponent.h"
#include "FPSProjectile.h"
#include "FPSCharacter.generated.h"

UCLASS()
class BAKUEATSNIGHTMARES_API AFPSCharacter : public ACharacter
{
   GENERATED_BODY()

public:
   // Sets default values for this character's properties
   AFPSCharacter();

protected:
   // Called when the game starts or when spawned
   virtual void BeginPlay() override;

   // 스폰할 발사체 클래스
   /* TSubclassOf는 AFPSProjectile을 상속받는 클래스를 저장할 수 있는 템플릿 클래스 타입이다.
   * 즉, ProjectileClass는 AFPSProjectile을 상속받는 클래스를 저장할 수 있는 변수인 것이다. (캡슐화)
   * protected에 지정해주는 이유 :
   * AFPSProjectile와 FPSCharacter 이 외의 클래스에서 AFPSProjectile을 상속받는 클래스에 접근하지 못하게 하기 위함 (무결성 유지, 상속을 통한 기능 확장)
   * ex) AFPSProjectile을 상속받은 AMyTile이 있다 했을 때 AMyTile 자신과 FPSCharacter와 FPSCharacter를 상속받은 클래스만 AMyTile 수정 및 접근 가능)
   */
   UPROPERTY(EditDefaultsOnly, Category = Projectile)
   TSubclassOf<class AFPSProjectile> ProjectileClass;

public:   
   // Called every frame
   virtual void Tick(float DeltaTime) override;

   // Called to bind functionality to input
   virtual void SetupPlayerInputComponent(class UInputComponent* PlayerInputComponent) override;

   // 앞뒤 이동 입력 처리
   UFUNCTION()
   void MoveForward(float value);

   // 좌우 이동 입력 처리
   UFUNCTION()
   void MoveRight(float value);

   UFUNCTION()
   void StartJump();

   UFUNCTION()
   void StopJump();

   // FPS 카메라
   UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = Camera)
   UCameraComponent* FPSCameraComponent;

   // 일인칭 메시(팔), self player에게만 보임
   UPROPERTY(VisibleDefaultsOnly, Category = Mesh)
   USkeletalMeshComponent* FPSMesh;

   // 발사체 발사를 처리하는 함수
   UFUNCTION()
   void Fire();

   /*[offset]
   * 카메라 위치로붙어의 총구 offset
   * offset : 객체의 위치, 회전, 크기 등을 기준 위치에서 얼마나 이동했는지를 나타냄
   * 카메라 위치에서 총구가 앞으로 얼마나 나와있는지를 나타내는 백터를 뜻한다.
   * 로컬 좌표계에서 정의되며 카메라 위치를 기준으로 한 상대적인 위치를 나타낸다.
   */
   UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = GamePlay)
   FVector MuzzleOffset;
};

```

``` c
// Fill out your copyright notice in the Description page of Project Settings.


#include "FPSCharacter.h"

// Sets default values
AFPSCharacter::AFPSCharacter()
{
    // Set this character to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
   PrimaryActorTick.bCanEverTick = true;
   
   // 일인칭 카메라 컴포넌트 생성
   FPSCameraComponent = CreateDefaultSubobject<UCameraComponent>(TEXT("FirstPersonCamera"));
   check(FPSCameraComponent != nullptr);

   // 캡슐 컴포넌트에 카메라 컴포넌트 붙임
   FPSCameraComponent->SetupAttachment(CastChecked<USceneComponent, UCapsuleComponent>(GetCapsuleComponent()));

   // Pawn이 카메라 회전을 제어하도록 설정
   FPSCameraComponent->bUsePawnControlRotation = true;

   // self Player의 일인칭 메시 컴포넌트 생성
   FPSMesh = CreateDefaultSubobject<USkeletalMeshComponent>(TEXT("FirstPersonMesh"));
   check(FPSMesh != nullptr);



   // FPS 메시를 FPS 카메라에 붙임
   FPSMesh->SetupAttachment(FPSCameraComponent);

   // 일부 섀도를 비활성화 하여 단일 메시 느낌 줌
   FPSMesh->bCastDynamicShadow = false;
   FPSMesh->CastShadow = false;


}

// Called when the game starts or when spawned
void AFPSCharacter::BeginPlay()
{
   Super::BeginPlay();
   
   if (FPSCameraComponent) {
      // 카메라가 플레이어의 눈 약간 위에 위치하도록 조정
      FPSCameraComponent->SetRelativeLocation(FVector(0.0f, 0.0f, 50.0f + BaseEyeHeight));
   }

   // self Player만 이 메시를 볼수 있도록 설정
   FPSMesh->SetOnlyOwnerSee(true);
   // self Player에게 본체가 되는 기본 메시가 보이지 않도록 설정
   GetMesh()->SetOwnerNoSee(true);
}

// Called every frame
void AFPSCharacter::Tick(float DeltaTime)
{
   Super::Tick(DeltaTime);

}

// Called to bind functionality to input
void AFPSCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
   Super::SetupPlayerInputComponent(PlayerInputComponent);
   
   // 'movement' 바인딩 구성
   PlayerInputComponent->BindAxis("MoveForward", this, &AFPSCharacter::MoveForward);
   PlayerInputComponent->BindAxis("MoveRight", this, &AFPSCharacter::MoveRight);

   // 'look' 바인딩 구성
   PlayerInputComponent->BindAxis("Turn", this, &AFPSCharacter::AddControllerYawInput);
   PlayerInputComponent->BindAxis("Look", this, &AFPSCharacter::AddControllerPitchInput);

   // 'action' 바인딩 구성
   PlayerInputComponent->BindAction("Jump", IE_Pressed, this, &AFPSCharacter::StartJump);
   PlayerInputComponent->BindAction("Jump", IE_Released, this, &AFPSCharacter::StopJump);
   PlayerInputComponent->BindAction("Firing", IE_Pressed, this, &AFPSCharacter::Fire);

}

void AFPSCharacter::MoveForward(float value)
{
   // 어디가 '앞'인지 찾고, 플레이어가 해당 방향으로 이동하고자 한다는 것을 기록합니다.
   FVector Direction = FRotationMatrix(Controller->GetControlRotation()).GetScaledAxis(EAxis::X);
   // 플레이어를 앞뒤로 이동
   AddMovementInput(Direction, value);
}

void AFPSCharacter::MoveRight(float value)
{
   // 어디가 '오른쪽'인지 찾고, 플레이어가 해당 방향으로 이동하고자 한다는 것을 기록합니다.
   FVector Direction = FRotationMatrix(Controller->GetControlRotation()).GetScaledAxis(EAxis::Y);
   // 플레이어를 좌우로 이동
   AddMovementInput(Direction, value);
}

void AFPSCharacter::StartJump()
{
   bPressedJump = true;
}

void AFPSCharacter::StopJump()
{
   bPressedJump = false;
}

void AFPSCharacter::Fire()
{
   // 발사체 발사 시도
   if (ProjectileClass)
   {
      FVector CameraLocation;
      FRotator CameraRotation;
      // 캐릭터의 카메라 위치와 방향의 위치와 회전을 가져옴
      GetActorEyesViewPoint(CameraLocation, CameraRotation);

      // MuzzleOffset이 카메라 살짝 앞에서 발사체를 스폰하도록 설정
      // 로컬 좌표
      MuzzleOffset.Set(100.0f, 0.0f, 0.0f);

      /* MuzzleOffset을 World Space로 변경
      * 왜 Rotation을 더해주는가?
      * -> MuzzleOffset.Set(100.0f, 0.0f, 0.0f)으로 카메라 앞에 100 더해서 위치가 있지만 이는 방향이 포함되지 않은 값이다.
      * 그래서 단지 거리만을 더한다고해서 캐릭터가 바라보는 방향과 일치하게 +100.0f가 되는게 아닌것이다.
      * 그래서 World Space로 변경해줄 때 방향도 같이 지정해 주어야 하는 것이다.
      */
      FVector MuzzleLocation = CameraLocation + FTransform(CameraRotation).TransformVector(MuzzleOffset);

      // 조준이 살짝 위를 향하도록 방향을 잡아준다.
      FRotator MuzzleRotation = CameraRotation;
      MuzzleRotation.Pitch += 10.0f;

      // 현재 레벨(월드)를 가져옴
      UWorld* World = GetWorld();
      if (World) {
         // 액터 생성(스폰) 시 특정 동작을 정의할 수 있는 구조체
         FActorSpawnParameters SpawnParams;
         // 액터의 소유자
         SpawnParams.Owner = this;
         // 이벤트 유발자를 추적함 
         // ex) 상대방에게 피해를 입혔을 때 피해를 입힌 캐릭터가 인스게이터가 된다.
         // 그럼 인스게이터(행위 유발자)에게 보상이나 점수를 줄 수 있게 된다.
         SpawnParams.Instigator = GetInstigator();

         AFPSProjectile* Projectile = World->SpawnActor<AFPSProjectile>(ProjectileClass, MuzzleLocation, MuzzleRotation, SpawnParams);
         if (Projectile) {
            FVector LaunchDirection = MuzzleRotation.Vector();
            // FireInDirection : projectile을 특정 방향으로ㅓ 발사하는 기능을 구현한 함수
            Projectile->FireInDirection(LaunchDirection);
         }
      }
   }
}

```