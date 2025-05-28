AddToViewPort (350p)
```cpp title:Controller.cpp
void BeginPlay(){
	AddToViewPort();
}
```

GameController(685p)
```cpp title:GameController.cpp
Pawn* PossessedPawn = EventOnPossess(); //Possessed Pawn을 반환
APlayerCharacter* Player = Cast<APlayerCharacter>(PossessedPawn);
Player->CreateWidget(this, Player);
CreateWidget->AddToViewport();
```

GameMode
-> 커스텀한 컨트롤러로 설정
