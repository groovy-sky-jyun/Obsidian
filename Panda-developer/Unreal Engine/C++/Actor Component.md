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
>생성자에서 호출해야 한다.
>생성자 외부에서 호출하려면 #NewObject 를 사용해야 한다.

> __에셋 레퍼런스__
> `"ObjectType'/Path/To/Asset.Asset'"` 를 넣어주면 된다.