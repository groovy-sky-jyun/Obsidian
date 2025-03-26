### Actor Component
#CreateDefaultSubobject 를 이용하여 서브오브젝터 클래스의 CDO 인스턴스를 생성한다.
```cpp title:MyActor.h
UPROPERTY()
UStaticMeshComponent* Mesh; 
```

```cpp title:MyActor.cpp
Mesh = CreateDefaultSubobject<UStaticMeshComponent>("BaseMeshComponent");

//#include "ConstructorHelpers.h"
auto MeshAsset = ConstructorHelpers::FObjectFinder<UStaticMesh>(TEXT("레퍼런스 주소"));
if(MeshAsset.object)
{
	Mesh->SetStaticMesh(MeshAsset.Object);
}
```
>##### #CreateDefaultSubobject
><span style="color:rgb(255, 192, 0)">생성자</span>에서 호출해야 한다.
>생성자 외부에서 호출하려면 #NewObject 를 사용해야 한다.

> __에셋 레퍼런스__
> `"ObjectType'/Path/To/Asset.Asset'"` 를 넣어주면 된다.

---
### Scene Component
액터 컴포넌트의 서브클래스이다.
상대 위치, 회전, 스케일 값을 결정하는 #transform 을 가지고 있다.

---
### OrbitingMovement Component
종속된 컴포넌트를 특정 방식으로 이동하도록 설계할 수 있다.
고정된 지점 주변의 궤도에 부착시킨다.
고정된 거리에서 이동시킨다.