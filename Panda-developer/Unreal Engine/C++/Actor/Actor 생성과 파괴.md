### Actor 클래스 인스턴스 생성
```cpp
Transform SpawnLocation;
GetWorld()->SpawnActor<AActor>(AActor::StaticClass(), SpawnLocation);
```

### Actor 삭제
`AActor->Destroy();`

### Actor 수명주기 설정
#### 1. 액터 생성 되자마자 수명 카운트
액터가 생성되자 마자 생명주기 카운트가 시작되도록 하려면 #InitialLifeSpan 변수에 시간을 지정해주면 된다.
액터가 생성되고, 지정된 시간이 끝나면 액터가 자신의 Destroy() 메서드를 호출한다.

```cpp
void MyActor::BeginPlay()
{
	Super::BeginPlay();
	InitialLifeSpan = 10.f; 
}
```
>__InitialLifeSpan 작성 위치__
>수명주기를 설정해주기 위해서는 BeginPlay() 메서드 안에서 설정해주어야 한다. 

<br>

#### 2. 수동 수명 카운트
코드 진행 도중에 특정 시간 딜레이를 준 뒤 Destroy()를 호출하고 싶을 때 #SetLifeSpan 을 사용한다.
해당 코드가 실행되고 나서 지정된 시간이 지나면 액터가 자신의 Destroy() 메서드를 호출한다.

```cpp
void MyActor::MyFunction()
{
	UE_LOG(LogTemp, Warning, TEXT("Count Start"));
	SetLifeSpan(10); 
}
```
>__SetLifeSpan 작성 위치__
>꼭 BeginPlay()가 아니어도 된다. 원하는 위치에 해당 함수를 작성하면 된다. 
>Destroy() 를 몇 초 뒤에 실행시키고 싶을 때 Timer를 사용하는 대신 간단하게 사용할 수 있다.


