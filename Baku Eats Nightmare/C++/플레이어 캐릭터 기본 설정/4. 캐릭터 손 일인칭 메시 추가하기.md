<br>

### 주제

주제설명
<br>

### <span style="color:rgb(205, 174, 234)">[FPSCharacter.h]</span>
#### <span style="color:rgb(243, 211, 155)">18</span> : 부제목

``` c
USkeletalMeshComponent* FPSMesh;
```

코드설명
<br>

### 전체 코드
``` c title:FPSCharacter.h  hl:17-18
...
UCLASS() 
class BAKUEATSNIGHTMARES_API AFPSCharacter : public ACharacter
{
	...
	
public:	
	// Called every frame
	virtual void Tick(float DeltaTime) override;

	// Called to bind functionality to input
	virtual void SetupPlayerInputComponent(class UInputComponent* PlayerInputComponent) override;
	
	 ...
	 
	 // 일인칭 메시(팔), self player에게만 보임
	UPROPERTY(VisibleDefaultsOnly, Category = Mesh)
	USkeletalMeshComponent* FPSMesh;
};
```

--- 

### <span style="color:rgb(205, 174, 234)">[FPSCharacter.cpp]</span>
### 주제목

#### <span style="color:rgb(243, 211, 155)">8-16</span> : 부제목

``` c
FPSMesh = CreateDefaultSubobject<USkeletalMeshComponent>(TEXT("FirstPersonMesh"));
check(FPSMesh != nullptr);
```

 #CreateDefaultSubobject 로 현재 클래스에 #USkeletalMeshComponent 소유권을 부여하여 FPSMesh 변수를 통해 스켈레탈 메시를 관리할 수 있도록 한다. 그 뒤 #check 를 통해 생성 유무를 확인한다.
  _[[3. 캐릭터 카메라 설정]] 참고
<br>

``` c
FPSMesh->SetupAttachment(FPSCameraComponent);
```

위는 <span style="color:rgb(205, 174, 234)">*FPSCameraComponent에 FPSMesh를 부착*</span>하는 코드이다. #UCameraComponent 와 #USkeletalMeshComponent 전부 #USceneComponet 를 상속받기 때문에 타입이 일치하여 캐스팅이 필요하지 않는다. )
<br>

``` c
~~FPSMesh->bCastDynamicShadow = false;~~
FPSMesh->CastShadow = false;
```

~~#bCastDynamicShadow 는 동적 그림자의 활성화를 설정할 수 있다. false 로 비활성화 해주어 실시간으로(동적으로) FPSMesh의 그림자가 생성되지 않는다.~~

#CashShadow 는 모든 그림자의 활성화를 설정할 수 있다. false로 비활성화 해주어 FPSMesh의 어떠한 그림자도 생성되지 않도록 해준다.
<br>

#### <span style="color:rgb(243, 211, 155)">25-27</span> : 부제목

``` c
FPSMesh->SetOnlyOwnerSee(true);
```

#SetOnlyOwnerSee 는 해당 메시 소유자에게만 보아도록 설정하는 함수이다. 다른 플레이어는 이 메시를 볼 수 없다. 
Default값은 false로 모든 플레이어가 해당 메시를 볼 수 있다.
<br>

``` c
GetMesh()->SetOwnerNoSee(true);
```
#GetMesh 를 통해 현재 캐릭터의 메시(몸 전체, 자기자신)를 가져온다. 그 뒤 #SetOwnerNoSee 함수로 해당 메시를 플레이어에게 보이지 않도록 설정한다. 그럼 플레이어는 카메라에 붙어있는 팔 메시만 보이게 된다.
<br>

### 전체 코드
``` c title:FPSCharacter.cpp hl:8-16,25-27
#include "FPSCharacter.h" 

AFPSCharacter::AFPSCharacter()
{
	...
 
	// self Player의 일인칭 메시 컴포넌트 생성
	FPSMesh = CreateDefaultSubobject<USkeletalMeshComponent>(TEXT("FirstPersonMesh"));
	check(FPSMesh != nullptr);

	// FPS 메시를 FPS 카메라에 붙임
	FPSMesh->SetupAttachment(FPSCameraComponent);

	// 일부 섀도를 비활성화 하여 단일 메시 느낌 줌
	~~FPSMesh->bCastDynamicShadow = false;~~
	FPSMesh->CastShadow = false;
}

// Called when the game starts or when spawned
void AFPSCharacter::BeginPlay()
{
	...

	// self Player만 이 메시를 볼수 있도록 설정
	FPSMesh->SetOnlyOwnerSee(true);
	// self Player에게 본체가 되는 기본 메시가 보이지 않도록 설정
	GetMesh()->SetOwnerNoSee(true);
}

```

---
