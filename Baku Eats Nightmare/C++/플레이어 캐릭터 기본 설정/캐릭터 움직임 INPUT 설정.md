<br>

## 기능 설명

###### 캐릭터의 기본 동작 INPUT  설정

기본으로 제공되는 Character 클래스를 상속받을 것이기 때문에 _<span style="color:rgb(193, 173, 240)">부모 클래스를 Character로 지정</span>_ 한 후 c++ 파일을 생성해준다.
<br>

### <span style="color:rgb(135, 75, 195)">[FPSCharacter.h]</span>
##### <span style="color:rgb(217, 152, 99)">19-34</span> : <mark style="background: #FFB86CA6;">캐릭터 Input 함수 정의</mark>

``` c title:FPSCharacter.h  hl:19-34
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

##### virtual void SetupPlayerInputComponent(class UInputComponent* PlayerInputComponent) override; 
- 플레이어의 INPUT을 관리하는 함수이다. 키보드, 마우스, 컨트롤러 등의 값이 들어오면 어떻게 처리할 지를 정의한다.
- #virtual 은 재정의 할 수 있는 함수를 의미하며 자식 클래스에서 재정의된 함수가 실행된다.
- #UInputComponent 클래스는 특정 INPUT을 받으면 해당 INPUT와 연결된 특정 함수나 행동이 호출될 수 있도록 INPUT와 함수를 미리 바인딩 해준다.

##### MoveForward(float value);
- 캐릭터 <span style="color:rgb(193, 173, 240)">_앞뒤_</span> INPUT을 받았을 때 실행되는 함수이다.
- 매개변수로 INPUT을 누른 정도(초)를 받는다.

##### MoveRight(float value);
- 캐릭터 <span style="color:rgb(193, 173, 240)">_좌우_</span> INPUT을 받았을 때 실행되는 함수이다.
- 매개변수로 INPUT을 누른 정도(초)를 받는다.

##### StartJump();
- 캐릭터 <span style="color:rgb(193, 173, 240)">_점프_</span> INPUT을 받았을 때 실행되는 함수이다.
- <span style="color:rgb(255, 0, 0)">캐릭터가 상승한다.</span>

##### StopJump();
- 캐릭터 <span style="color:rgb(193, 173, 240)">_점프_</span> INPUT을 받았을 때 실행되는 함수이다.
- <span style="color:rgb(255, 0, 0)">캐릭터가 하강한다.</span>

<br>

> ##### 왜 함수 위에 #UFUNCTION 을 사용하는가?
> 함수위에 `UFUNCTION()`  적지 않아도 함수가 동작하는데에는 문제가 없다. 하지만 적지 않는다면 
> - 블루프린트에서 해당 함수를 사용할 수 없게된다. 
> - 네트워크 복제가 작동하지 않아 서버와 클라이언트 간의 통신 오류가 발생할 수 있다.

<br>

>#####  #UFUNCTION 은 어떤 기능을 제공하는가?
>- Blueprint 관련 속성
>- 네트워킹 관련 속성
이 외에도 있지만 지금은 이정도만 알고있도록 하자

<br>

---

### <span style="color:rgb(135, 75, 195)">[FPSCharacter.cpp]</span>
#### Input 함수 정의

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

#### SetupPlayerInputComponent
이 함수에서 사용되는 기본 Input 바인딩 구조는 아래와 같다.
1. 언리얼에서 `MoveForward` 라는 이름의 Input을 지정해 준다. 
2. 이를 this (현재 코드가 적힌 클래스를 사용하는 캐릭터) 에게 적용시킨다. 
3. 해당 키가 눌렸을 때 <span style="color:rgb(168, 217, 255)">AFPSCharacter</span> 클래스의 <span style="color:rgb(146, 208, 80)">MoveForward</span> 함수를 실행한다

이를 이용해 상/하, 좌/우, 캐릭터 시야(상/하, 좌/우), 점프 INPUT에 대한 함수를 바인딩해준다. 

<br>

> #### #BindAxis 와 #BindAction의 차이
> ##### <mark style="background: #ADCCFFA6;">BindAxis</mark>
> 플레이어가 특정 키를 입력하면 이벤트가 발생하여 키값을 바인딩하게된다.
> ##### <mark style="background: #ADCCFFA6;">BindAction</mark>
> BindAxis와 같은 기능을 제공하는데 차이점이 있다. 바로 키를 눌렀을 때와 땠을때의 동작을 다르게 설정할 수 있는 점이다. 이는 BindAction이 EInputEvent를 사용하기 때문에 가능한것이다. 
> EInput은 IE_Pressed와 IE_Released 두가지를 제공하고 있다.
> 이를 사용할 때에는 한가지 이름(언리얼 편집기에 지정된 Input 이름)에 대해 바인딩 함수를 2개(IE_Pressed, IE_Released)를 지정해주어야 한다.
>  -  IE_Pressed : 눌렀을 떄 동작 
>  -  IE_Released : 땠을 때 동작

---
##### <span style="color:rgb(217, 152, 99)">45-46</span> : <mark style="background: #FFB86CA6;">UObject </mark>
UCameraComponent 클래스의 포인터 변수를 선언한다.
>#### 왜 포인터로 선언하는가?
>#UObject 기반의 클래스의 경우 언리얼의 #GarbageCollectionSystem 과 관련이 있어 보통 포인터로 선언된다. 
>
>#GarbageCollectionSystem 은 포인터를 추적하여 더이상 참조되지 않는 객체를 자동으로 해제한다. 그러므로써 개발자가 직접 메모리를 할당하고 해제해야하는 부담을 줄여준다.  
>
>#UPROPERTY 매크로를 통해 #GarbageCollectionSystem 이 해당 객체를 추적할 수 있도록 해준다. 만약 #UObject 를 #UPROPERTY 로 선언하지 않으면  #GarbageCollectionSystem 이 해당 객체를 추적하지 않게 되어 메모리 누수 등의 문제가 발생할 수 있다.
>
