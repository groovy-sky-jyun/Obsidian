특정 객체의 개수가 너무 많을 때 좀 더 가볍게 만들 수 있는 패턴이다.

공유 상태 : 모든 객체가 공통으로 가지는 정보, 메모리 절약을 위해 공유
비공유 상태 : 개별 객체마다 달라지는 정보, 외부에서 주입

이렇게 객체가 공통으로 가지는 데이터를 공유하여 메모리 사용량을 줄이는 패턴이다.


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
- 타입별 속성이 하드코딩되어있다. -> 새로운 지형 추가 시 조건문을 전부 수정해 주어야 한다.
- 확장에 취약하다.
- 데이터 타입과 속성(행동)이 분리되어 있다. 각 타입과 getMovementCost같은 속성이 다른 클래스에 정의되어 있다. -> 객체지향적이지 않다.

---

```cpp
class Tarrain{
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

class World{
private:
	Terrain* tiles_[Width][Height];
};

Terrain* grass = new Terrain(1, false, grassTexture);
Terrain* hill = new Terrain(2, false, hillTexture);
Terrain* river = new Terrain(3, true, riverTexture);

tiles_[0][0] = grass;
tiles_[0][1] = hill;
tiles_[0][2] = grass; // 또 grass
tiles_[0][3] = river;
```
- Terrian 클래스가 속성과 기능을 함께 소유하고 있다. -> 객체지향적
- tiles_ 배열에 Terrian 객체 포인터만 저장
	- 동일한 지형

**장점**
- Terrain에 새로운 속성을 추가하더라도 해당 클래스만 수정하면 된다.
- 동일한 속성의 Terrain은 한 번만 생성하고, 여러 타일에서 참조할 수 있다. -> 메모리 절약
