## 구현 알고리즘
1) 벽 네개가 온전한 오브젝트를 준비한다. (AWall 이라고 하겠다.)
2) AWall을 Maze 크기에 맞게 Grid로 배치한다.
3) DFS 알고리즘을 활용하여 미로를 생성한다.
4) 입구와 출구를 지정해준다.

### AWall 오브젝트


### 미로 초기화
- 미로의 Height, Width 크기만큼 AWall을 Spawn 한다.
- 생성된 AWall 클래스는 MazeGrid[x][y]에 담아준다.
- AWall의 IndexX와 IndexY의 값을 지정해준다. 이는 각각 스폰된 AWall 클래스가 담긴 MazeGrid의  인덱스 번호를 뜻한다. 
``` cpp unwrap:false title:MazeGenerator.cpp hl:14-15
// 미로 초기화(네 면이 다 있는 Wall 격자 생성)
void AMazeGenerator::GenerateMaze()
{ 
    FVector BaseLocation = GetActorLocation();
    FRotator Rotation = GetActorRotation();
    //벽이 전체 온전한 미로 생성 (초기화)
    for (int x = 0; x < Width; x++) {
        for (int y = 0; y < Height; y++) {
            //벽 스폰 위치 설정
            FVector Location = BaseLocation + FVector(x * 400.0f, y * 400.0f, 0.0f);
            //벽 스폰
            AWall* SpawnWall = GetWorld()->SpawnActor<AWall>(BP_Wall, Location, Rotation);
            if (SpawnWall) { 
                SpawnWall->IndexX = x;
                SpawnWall->IndexY = y;
                MazeGrid[x].Row[y] = SpawnWall; 
            }
            else {
                UE_LOG(LogTemp, Error, TEXT("Failed to spawn AWall Actor"));
            }
        }
    }
}
```

``` cpp title:Wall.cpp
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Wall")
	int IndexX; 

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Wall")
	int IndexY;
```

### 미로 생성
1. 시작지점(0,0) 부터 (방문하지 않았으면서 && 현재 큐브와 인접한곳) 중 랜덤으로 한군데씩 방문한다.
	1. 현재 큐브를 중심으로 상하좌우를 탐색하여 방문하지 않은 큐브들을 List에 담는다.
	2. List에서 랜덤으로 큐브하나를 뽑는다. (NeighborWall 이라 칭하겠다.)
	3. NeighborWall와 현재 큐브 사이의 벽을 제거해준다.
	4. NeighborWall로 위치를 이동한다. (뽑힌 큐브가 현재 큐브가 된다.)
	5. 1의 조건이 만족할 때 까지 1.1~1.4을 반복한다.
2. (방문하지 않았으면서 && 인접한 큐브)가 한개도 없다면 방문했던 큐브를 거슬러 가면서 해당 큐브 주변에 방문하지 않은 큐브가 남아있는지 확인한다.
	1. 방문했던 큐브들을 순서대로 Stack에 넣는다.
	2. 현재 큐브 중심으로 방문하지 않은 큐브가 없다면 pop을 한다.
	3. Stack의 Top에 있는 큐브를 중심으로 방문하지 않은 큐브가 있는지 확인한다.
	4. 방문하지 않은 큐브가 있다면 해당 큐브를 Stack에 넣고 해당 큐브가 현재 큐브가 된다. 
		1. 1.1~1.4을 진행한다.
	5. 방문하지 않은 큐브가 없다면 2.2로 돌아간다.
``` cpp unwrap:false title:MazeGenerator.cpp_MakePassages() hl:17,31,32,35,37
void AMazeGenerator::MakePassages(int x, int y)
{
    /*
    * [방문하지 않은 곳이라면] 
    * Stack에 Add   
    * 방문 = true
    */
    if (!CheckList[x].bRow[y]) {  
        Stack.Add(MazeGrid[x].Row[y]); 
        CheckList[x].bRow[y] = true;
    }
    
    /*
    * [주변 방문하지 않은 큐브들 List 가져오기]
    */
    TArray<AWall*> FourWallList;
    FourWallList = CheckVisitList(x, y, FourWallList); 
   
    /*
    * [주변 방문하지 않은 큐브가 하나라도 있다면]
    * 1) List 에서 랜덤으로 한개 큐브 뽑음
    * 2) 랜덤 큐브와 현재 큐브 사이의 벽 제거
    * 
    * [주변 방문하지 않은 큐브가 하나라도 없다면]
    * 1) Stack 에서 Pop 한다.
    * 2) Stack 의 Top 큐브 주변에 방문하지 않은 큐브가 있는지 확인한다.
    */
    if (FourWallList.Num() > 0) {

        int32 ShuffleNum = FMath::RandRange(0, FourWallList.Num() - 1);
        DeleteWall(MazeGrid[x].Row[y], FourWallList[ShuffleNum]);
        MakePassages(FourWallList[ShuffleNum]->IndexX, FourWallList[ShuffleNum]->IndexY);
    }
    else {
        AWall* PopCube = Stack.Pop();
        if (Stack.Top()->IndexX != 0 || Stack.Top()->IndexY != 0) {
            MakePassages(Stack.Top()->IndexX, Stack.Top()->IndexY);
        }
    }
}
```

``` cpp unwrap:false title:MazeGenerator.cpp_CheckVisitList()
TArray<AWall*> AMazeGenerator::CheckVisitList(int x, int y, TArray<AWall*> FourWallList)
{
	/*
	* 현재 큐브를 기준으로 상하좌우 큐브에 방문했는지 확인후
	* 방문하지 않았다면 List에 담는다.
	* 방문하지 않은 AWall을 담은 배열을 return한다.
	*/

    //위측 블럭 방문 확인 
    if (y + 1 < Height) {
        if (FourWallList.Num() < 4 && !CheckList[x].bRow[y+1]) { 
            FourWallList.Add(MazeGrid[x].Row[y+1]);
        }
    }

    //아래측 블럭 방문 확인 
    if (y - 1 >= 0) {
        if (FourWallList.Num() < 4 && !CheckList[x].bRow[y-1]) { 
            FourWallList.Add(MazeGrid[x].Row[y-1]);
        }
    }

    //우측 블럭 방문 확인 
    if (x + 1 < Width) {
        if (FourWallList.Num() < 4 && !CheckList[x+1].bRow[y]) { 
            FourWallList.Add(MazeGrid[x + 1].Row[y]);
        }
    }

    //좌측 블럭 방문 확인 
    if (x - 1 >= 0) {
        if (FourWallList.Num() < 4 && !CheckList[x-1].bRow[y]) {
            FourWallList.Add(MazeGrid[x-1].Row[y]);
        }
    }

    return FourWallList;
}
```

``` cpp unwrap:false title:MazeGenerator.cpp_DeleteWall()
void AMazeGenerator::DeleteWall(AWall* OriginWall, AWall* NeighborWall) 
{
	/*
	* 현재 큐브와 랜덤으로 뽑힌 큐브 사이의 벽을 제거한다.
	*/
	
    int x = OriginWall->IndexX; 
    int y = OriginWall->IndexY;
    int nx = NeighborWall->IndexX;
    int ny = NeighborWall->IndexY;

    if (x - nx < 0) { //(0,0)(1,0) 위쪽 큐브 선택 (랜덤큐브)
        OriginWall->DeleteOnceWall("Front"); 
        NeighborWall->DeleteOnceWall("Bottom");
    }
    else if (x - nx > 0) {//(1,0)(0,0) 아래쪽 큐브 선택 (랜덤큐브)
        OriginWall->DeleteOnceWall("Bottom"); 
        NeighborWall->DeleteOnceWall("Front"); 
    }
    else if (y - ny < 0) {//(0,0)(0,1) 오른쪽 큐브 선택 (랜덤큐브)
        OriginWall->DeleteOnceWall("Right"); 
        NeighborWall->DeleteOnceWall("Left"); 
        
    else if (y - ny > 0) {//(0,1)(0,0) 왼쪽 큐브 선택 (랜덤큐브)
        OriginWall->DeleteOnceWall("Left"); 
        NeighborWall->DeleteOnceWall("Right"); 
    }

}
```

### 입구/출구 지정
``` cpp unwrap:false title:MazeGenerator.cpp_BeginPlay() hl:24-25
void AMazeGenerator::BeginPlay() 
{
	Super::BeginPlay();
	
    // ArrWallClass 크기 초기화
    MazeGrid.SetNum(Width);
    for (int i = 0; i < MazeGrid.Num(); i++) {
        MazeGrid[i].Row.SetNum(Height);
    }

    // CheckList Class 크기 초기화
    CheckList.SetNum(Width);
    for (int i = 0; i < CheckList.Num(); i++) {
        CheckList[i].bRow.SetNum(Height);
    }

    // 미로 초기화 - 생성
    GenerateMaze();

    // 길만들기
    MakePassages(0,0);

    // 입구, 출구
    MazeGrid[0].Row[0]->DeleteOnceWall("Bottom"); //입구
    MazeGrid[Width-1].Row[Height-1]->DeleteOnceWall("Front"); //출구

}
```

## 결과
![[스크린샷 2024-10-21 192759.png]]