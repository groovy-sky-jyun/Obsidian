


bp에서 인스턴스 생성
begin play -> Contruct Object from Class Blueprint 노드를 통해 인스턴스를 생성할 수 있다.
Outer로 소유자를 지정해 주어야 하는데 Self로 해주면 된다.(Get a reference to self)
Return value로 해당 인스턴스를 변수로 set 해줄 수 있다.(Promote to a variable)

이렇게 생성한 인스턴스를 c++에서 사용하는 방법이 있다.
바로 TSubclassOf, FSoftClassPath를 사용하는 것이다. 이는 드롭다운 메뉴를 통해 모든 블루프린트를 나열해준다. 즉, 블루프린트로 생성된 인스턴스를 참조할 때 사용한다.

- TSubclassOf : 해당 클래스에서 파생된 모든 Class를 출력
- FSoftClassPath : meta Class에서 파생된 블루프린트를 나타낸다. 프로젝트의 모든 블루프린트를 보려한다면 MetaClass를 그냥 두면 된다.

UObject에서 파생된 클래스를 인스턴스화 하는 방법
- NewObject< >
인스턴스화 할때는 인스턴스화 할 클래스 타입에 대한 c++ 타입의 UCLASS 참조(bp)
와 bp 클래스가 파생되는 원래의 c++ 베이스 클래스를 알아둬야한다.

.h파일에서 아래처럼 UCLASS 참조를 설정해준다. 
``` cpp title:UObject.h
UPROPERTY(EditAnywhere, BlueprintReadWrite)
TSubclassOf<UUserProfile> UClassReference;
```
.cpp 파일에서 아래처럼 오브젝트를 인스턴스화해준다.
``` cpp title:UObject.cpp
UUserProfile* UserProfile = NewObject<UUserProfile>(UClassReference);
```

이를 이용하여 다른 클래스에서 접근하는것 해보기

