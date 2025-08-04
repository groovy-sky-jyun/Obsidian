### 1. 타일 기반 맵
타일의 종류는 `잔디, 강, 산` 3가지가 있고, 총 1,000개의 타일이 필요하다 할 때, 이를 생성하는 코드를 작성해보자.

--- 
#### Flywight Pattern 미적용
```cpp
enum Terrian{     
Terrain_grass, 
Terrain_hill,   
Terrain_river
};

class World{
private:
	Terrain tiles_[Width][Height];
};

int World::getMovementCost(int x, int y){
	switch (tiles_[x][y]){
		case Terrain_grass: return 1;
		case Terrain_hill: return 2;
		case Terrain_river: return 3;
		
	} 
}

bool World::isWater(int x, int y){
	if(tiles_[x][y] == Terrain::Terrain_river)
		return true; 
	else
		return false;
}
```
<br>

> **문제점**
> - 타입별 속성이 하드코딩되어있다. -> 새로운 지형 추가 시 조건문을 전부 수정해 주어야 한다.
> - 확장에 취약하다.
> - 객체지향적이지 않다.

---

#### Flyweight Pattern 적용
Flyweight 패턴을 사용하면 종류별 타일(grass, river, hill)에 해당하는 Terrain **객체를 한 번만 생성**하고, 이를 참조하여 맵의 모든 타일에 적용함으로써 메모리를 절약할 수 있다.

##### 1. 공유 상태 정의
```cpp
class Terrain{  
public:
	Terrain(int movementCost, bool isWater, Texture texture)
	: movementCost_(movementCost),
	isWater_(isWater),
	texture_(texture)

	int getMovementCost() const { return movementCost_; }
	bool getIsWater() const { return isWater_; }
	Texture getTexture() const { return texture_; }

private:
	int movementCost_;
	bool isWater_;
	Texture texture_;
};
```
- Terrain 클래스는 모든 각 타입의 타일이 공통으로 가지는 속성(<span style="color:rgb(255, 192, 0)">공유상태</span>_예)이동 비용, 물 여부, 텍스처 등)을 보관한다.


##### 2. 월드에 베이스 객체 생성 및 타일 참조
```cpp
class World{
public:
	World()
	: grassTerrain_(1, false, GRASS_TEXTURE),
	  hillTerrain_(3, false, HILL_TEXTURE),
	  riverTerrain_(2, true, RIVER_TEXTURE){
		  generateTerrain();
	  }

	const Terrain& World::getTile(int x, int y) const{
		return *tiles_[x][y];
	}

private:
	static constexpr int Width = 100;
	static constexpr int Height = 100;
	
	Terrain* tiles_[Width][Height];
	Terrain grassTerrain_;
	Terrain hillTerrain_;
	Terrain riverTerrain_;
	
	void generateTerrain(){
	//1) 기본 지형 생성성: 랜덤으로 grass, hill 참조
	for(int x = 0; x<Width; x++){
		for(int y = 0; y < Height; y++){
			if(random(10) ==0){
				tiles_[x][y] = &hillTerrain_;
			}else{
				tiles_[x][y] = &grassTerrain_;
			}
		}
	}
	//2) 강 지형: 랜덤 열 전체에 river 참조
	int x = random(Width);
	for(int y = 0; y < Height; y++){
		tiles_[x][y] = &riverTerrain_;
	}
}
	  

```

- tiles_[x][y] 배열 인덱스는 <span style="color:rgb(255, 192, 0)">비공유 상태</span>로, 각 타일의 위치를 나타낸다.
- 모든 tiles_ 요소는 <span style="color:rgb(255, 192, 0)">기존에 생성된 Terrain 객체를 참조</span>하므로, <span style="color:rgb(255, 192, 0)">추가 메모리 할당 없이</span> 수천개의 타일을 효율적으로 관리할 수 있다.
- 
<br>

> **장점**
> - Terrain에 새로운 속성을 추가하더라도 해당 클래스만 수정하면 된다.
> - 동일한 속성의 Terrain은 한 번만 생성하고, 여러 타일에서 참조할 수 있다. -> 메모리 절약

---
