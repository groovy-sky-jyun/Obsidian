UClass는 메타클래스로 클래스 자체를 나타내는 메타정보를 가진다. 이를 사용하면 클래스가 UE의 스마트 포인터와 메모리 관리 루틴을 사용하여 인스턴스를 생성하고 동작을 정의하게된다.
C++은 new 와 delete 연산자를 사용해 커스텀 오브젝트를 생성하고 삭제하지만 UE에서는 UCLASS 매크로를 사용하여야 한다.

Object의 Instance 생성
- UObject에서 파생된 클래스를 인스턴스화 할때 NewObject 함수 사용
- 그 외에서 파생된 클래스를 인스턴스화 할때 UWorld::SpawnActor< > 함수 사용

Object의 Instance 삭제
UObject::ConditionalBeginDestroy() 함수를 사용한다.

UObject와 Blueprint 관계
Uobject에서 파생된 오브젝트는 레벨에 배치할 수 없다. 하지만 Blueprint로는 만들 수 있다. 
1. c++에서 UCLASS(Blueprintable, BlueprintType) 을 정의해준다.
2. Window > Developer Tools > Class Viewer 에서 Filterse > Actors Only 설정을 끈다.
3. c++파일에서 우클릭하여 create Blueprint 버튼을 사용한다.
(Blueprintable만 사용해도 되는지 확인!)

UCLASS(키워드)
키워드에 들어갈 수 있는것은 다음과 같다.
- UCLASS(Blueprintable)
	- Class Viewer에서 해당 클래스를 BP로 생성하고자 할 때 사용. 해당 키워드를 사용해야 Create Blueprint Class... 옵션을 사용할 수 있다.
- UCLASS(BlueprintType)
	- 다른 BP 에서 변수처럼 사용할 수 있다. 
- UCLASS(NotBlueprintType)
	-  BP를 만들 수 없다.


UPROPERTY()
UPROPERTY() 는 변수를 선언할 때 필요하며 전달되는 파라미터는 변수에 관한 정보를 지정한다.
- EditAnywhere 
	- BP, 게임 레벨 내 각uclass 오브젝트 인스턴스에서 수정 가능
- EditDefaultsOnly 
	- BP 편집 가능하지만 인스턴스 단위로는 수정 불가능. 게임 시작 전에 설정되는 값으로 런타임에는 편집할 수 없다.
- EditInstanceOnly 
	- 게임 레벨 인스턴스에서 속성을 편집할 수 있다.
- BlueprintReadWrite
	- 블루프린트가 해당 변수를 언제든 읽고 쓸 수 있다. public이여야 한다.
- BlueprintReadOnly 
	- c++에서만 설정 가능하고 bp에서 수정 불가능
- Category
	- 하위 메뉴 이름을 나타낸다. 카테고리를 지정하지 않으면 해당 클래스 이름 카테고리 영역에 표시된다.


