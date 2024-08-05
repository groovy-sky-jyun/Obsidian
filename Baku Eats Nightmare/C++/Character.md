https://dev.epicgames.com/documentation/ko-kr/unreal-engine/implementing-your-character-in-unreal-engine

MyCharacter.h

``` c++
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Character.h"
#include "GameFrameWork/SpringArmComponent.h"
#include "Camera/CameraComponent.h"
#include "MyCharacter.generated.h"

UCLASS()
class BAKUEATSNIGHTMARES_API AMyCharacter : public ACharacter
{
	GENERATED_BODY()

public:
	// Sets default values for this character's properties
	AMyCharacter();

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

public:	
	// Called every frame
	virtual void Tick(float DeltaTime) override;

	// Called to bind functionality to input
	virtual void SetupPlayerInputComponent(class UInputComponent* PlayerInputComponent) override;

private:
	void MoveForward(float Value);
	void MoveRight(float Value);
	void StartCrouch();
	void StopCrouch();
	void TurnAtRate(float Value);
	void LookUpAtRate(float Value);

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Movement", meta = (AllowPrivateAccess = "true"))
	float CrouchSpeed = 150.0f;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Movement", meta = (AllowPrivateAccess = "true"))
	float WalkSpeed = 600.0f;

	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Camera", meta = (AllowPrivateAccess = "true"))
	float BaseTurnRate = 45.0f;

	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Camera", meta = (AllowPrivateAccess = "true"))
	float BaseLookUpRate = 45.0f;

	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Camera", meta = (AllowPrivateAccess = "true"))
	USpringArmComponent* CameraBoom;

	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Camera", meta = (AllowPrivateAccess = "true"))
	UCameraComponent* FollowCamera;

};

```

MyCharacter.cpp
``` c++
// Fill out your copyright notice in the Description page of Project Settings.


#include "MyCharacter.h"
#include "GameFramework/CharacterMovementComponent.h"
#include "GameFramework/Controller.h"
#include "GameFramework/SpringArmComponent.h"
#include "Camera/CameraComponent.h"
#include "Components/InputComponent.h"


// Sets default values
AMyCharacter::AMyCharacter()
{
 	// Set this character to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;

	//Set default movement speeds
	GetCharacterMovement()->MaxWalkSpeed = WalkSpeed;

	//Create the camera boom
	CameraBoom = CreateDefaultSubobject<USpringArmComponent>(TEXT("CameraBoom"));
	CameraBoom->SetupAttachment(RootComponent);
	CameraBoom->TargetArmLength = 300.0f;
	//플레이어(Pawn)의 회전 컨트롤을 따라가도록 설정
	CameraBoom->bUsePawnControlRotation = true;

	//Create the follow camera
	FollowCamera = CreateDefaultSubobject<UCameraComponent>(TEXT("FollowCamera"));
	//CameraBoom에 부착하여 카메라가 스프링 암의 끝에 위치하도록 설정
	FollowCamera->SetupAttachment(CameraBoom, USpringArmComponent::SocketName);
	//스프링 암을 통해 간접적으로 회전하면 더 부드러운 카메라 움직임을 구현할 수 있다.
	FollowCamera->bUsePawnControlRotation = false;

	GetCharacterMovement()->GetNavAgentPropertiesRef().bCanCrouch = true;
}

// Called when the game starts or when spawned
void AMyCharacter::BeginPlay()
{
	Super::BeginPlay();
	
}

// Called every frame
void AMyCharacter::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

}

// Called to bind functionality to input
void AMyCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);

	PlayerInputComponent->BindAxis("MoveForward", this, &AMyCharacter::MoveForward);
	PlayerInputComponent->BindAxis("MoveRight", this, &AMyCharacter::MoveRight);
	//IE_Pressed : 키가 눌렸을 때, IE_Released : 키가 놓였을 때
	PlayerInputComponent->BindAction("Crouch", IE_Pressed, this, &AMyCharacter::StartCrouch);
	PlayerInputComponent->BindAction("Crouch", IE_Released, this, &AMyCharacter::StopCrouch);
	// 입력값에 따른 캐릭터 좌우 회전 처리
	PlayerInputComponent->BindAxis("Turn", this, &APawn::AddControllerYawInput);
	// 입력값에 따라 캐릭터 좌우 회전 속도 제어
	PlayerInputComponent->BindAxis("TurnRate", this, &AMyCharacter::TurnAtRate);
	// 입력값에 따른 캐릭터 상하 회전 처리
	PlayerInputComponent->BindAxis("LookUp", this, &APawn::AddControllerPitchInput);
	// 입력값에 따라 캐릭터 상하 회전 속도 제어
	PlayerInputComponent->BindAxis("LookUpRate", this, &AMyCharacter::LookUpAtRate);

}

void AMyCharacter::MoveForward(float Value)
{
	if ((Controller != nullptr) && (Value != 0.0f)) {

		//플레이어 시점 또는 카메라 방향 회전값 가져옴 
		const FRotator Rotation = Controller->GetControlRotation();

		/* 
		* [YawRotation(Pitch, Yaw, Roll)]
		*  Pitch : 위아래 회전 각도
		*  Yaw : 회전각도, 회전 축 -> 좌우 회전 각도
		*  Roll : 좌우 기울어짐
		*/
		const FRotator YawRotation(0, Rotation.Yaw, 0);


		//캐릭터가 전진할 방향 계산
		//X축 방향 벡터를 YawRotation 각도만큼 회전하겠다는 뜻
		const FVector Direction = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::X);

		//캐릭터 이동 함수
		AddMovementInput(Direction, Value);
	}
}

void AMyCharacter::MoveRight(float Value)
{
	if ((Controller != nullptr) && (Value != 0.0f)) {

		//플레이어 시점 또는 카메라 방향 회전값 가져옴 
		const FRotator Rotation = Controller->GetControlRotation();
		const FRotator YawRotation(0, Rotation.Yaw, 0);

		//Y축 방향 벡터를 YawRotation 각도만큼 회전하겠다는 뜻
		const FVector Direction = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::Y);

		AddMovementInput(Direction, Value);
	}
}

void AMyCharacter::StartCrouch()
{
	if (CanCrouch()) {//앉을 수 있는 상태인지 확인
		Crouch();
		GetCharacterMovement()->MaxWalkSpeed = CrouchSpeed;
	}
}

void AMyCharacter::StopCrouch()
{
	UnCrouch();
	GetCharacterMovement()->MaxWalkSpeed = WalkSpeed;
}

void AMyCharacter::TurnAtRate(float Rate)
{
	/*
		AddControllerYawInput : APawn 클래스의 함수로 캐릭터의 컨트롤러에 Yaw 회전 입력 추가
		Rate : 키를 얼마나 눌렀는지에 대한 수치 (오른쪽 회전-양수, 왼쪽 회전-음수)
		BaseTurnRate : 초당 회전 각도 (초당 최대 45도 회전)
		GetWorld()->GetDeltaSeconds() : 프레임 간의 시간 차이
	*/
	AddControllerYawInput(Rate * BaseTurnRate * GetWorld()->GetDeltaSeconds());
}

void AMyCharacter::LookUpAtRate(float Rate)
{
	AddControllerPitchInput(Rate * BaseLookUpRate * GetWorld()->GetDeltaSeconds());
}
```