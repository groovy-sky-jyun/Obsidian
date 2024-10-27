
<br>

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
	3. <span style="color:rgb(0, 176, 240)">pop 큐브와 인접하면서 && 방문한 큐브 리스트를 생성한다.</span>
	4. <span style="color:rgb(0, 176, 240)">리스트에서 0~1개 큐브를 선택한다.</span>
	5. <span style="color:rgb(0, 176, 240)"> pop을 한 큐브와 랜덤으로 선택된 큐브와의 벽을 허문다.</span>
	7. Stack의 Top이 있는 큐브로 이동한다.
	8. Stack의 Top에 있는 큐브를 중심으로 방문하지 않은 큐브가 있는지 확인한다.
	9. 방문하지 않은 큐브가 있다면 해당 큐브를 Stack에 넣고 해당 큐브가 현재 큐브가 된다. 
		- 1.1 로 돌아간다.
	10. 방문하지 않은 큐브가 없다면 2.1 로 돌아간다.
	
``` cpp unwrap:false title:MazeGenerator.cpp_MakePassages() hl:26,37
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
    * 2) Pop 큐브 주변에 방문한 큐브 중 한 큐브와 사이 벽 제거
    * 3) Stack 의 Top 큐브 주변에 방문하지 않은 큐브가 있는지 확인한다.
    */
    if (FourWallList.Num() > 0) {

        int32 ShuffleNum = FMath::RandRange(0, FourWallList.Num() - 1);
        DeleteWall(MazeGrid[x].Row[y], FourWallList[ShuffleNum]);
        MakePassages(FourWallList[ShuffleNum]->IndexX, FourWallList[ShuffleNum]->IndexY);
    }
    else {
        AWall* PopCube = Stack.Pop();
        AddPassages(PopCube);
        if (Stack.Top()->IndexX != 0 || Stack.Top()->IndexY != 0) {
            MakePassages(Stack.Top()->IndexX, Stack.Top()->IndexY);
        }
    }
}
```

``` cpp unwrap:false title:MazeGenerator.cpp_AddPassagest()
void AMazeGenerator::AddPassages(AWall* Cube)
{
    /*
    * [인접한 큐브 중 0-1개 뽑아서 벽을 허뭄] 
    * 1) 인접한 큐브 중 방문한 큐브 List 생성
    */
    TArray<AWall*> List;
    int x = Cube->IndexX;
    int y = Cube->IndexY;

    //현재 블럭의 위측 블럭 확인
    if (y + 1 < Height) {
        List.Add(MazeGrid[x].Row[y+1]);
    }
    //현재 블럭의 아래측 블럭 확인
    if (Cube->IndexY - 1 >= 0) {
        List.Add(MazeGrid[x].Row[y - 1]);
    }
    //현재 블럭의 우측 블럭 확인
    if (x + 1 < Width) {
        List.Add(MazeGrid[x+1].Row[y]);
    }
    //현재 블럭의 좌측 블럭 확인
    if (x - 1 >= 0) {
        List.Add(MazeGrid[x - 1].Row[y]);
    }

	/*
    * 2) 생성한 List 중 랜덤 0 또는 1개 큐브 뽑음
    * 3) 랜덤 큐브와 현재 큐브브 사이 벽 허뭄
    */
    if (List.Num() > 0) {
        int32 PickUp = FMath::RandRange(0, 1);
        if (PickUp > 0) {
            int32 ShuffleNum = FMath::RandRange(0, List.Num() - 1);
            DeleteWall(Cube, List[ShuffleNum]);
        }
    }
}
```

## 결과
![[스크린샷 2024-10-21 192550.png]]

![[스크린샷 2024-10-21 192637.png]]