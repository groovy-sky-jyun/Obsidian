### 2. Flyweight + Pooling
총알과 로켓을 발사하는 코드를 구현해보자.
총알과 로켓은 각각 damage, speed, texture를 다르게 가진다.

**Pooling와 함께 사용하는 이유**
> - <span style="color:rgb(255, 192, 0)">Flyweight로 공통 리소스는 공유</span>하고, <span style="color:rgb(255, 192, 0)">Pool로 인스턴스를 재활용</span>하면 <span style="color:rgb(128, 202, 255)">메모리, 퍼포먼스 모두 최적화</span>된다.
> - <span style="color:rgb(255, 192, 0)">Pool</span>만 사용하면 <span style="color:rgb(255, 192, 0)">new/delete</span>를 줄일 수 있지만, 각각의 객체가 데이터를 복사, 보관해야 하므로 <span style="color:rgb(255, 82, 82)">메모리가 많이 사용</span>된다.
> - <span style="color:rgb(255, 192, 0)">Flyweight</span>만 사용하면 <span style="color:rgb(255, 192, 0)">공통 데이터</span>는 절약된다. 허나 매번 new로 인스턴스를 만들면 <span style="color:rgb(255, 82, 82)">비용이 많이 든다.</span>

---

##### 1. 발사체 종류 Key
```cpp
enum class ProjectileKind { Bullet, Rocket };
```

---

##### 2. 공유 상태만 보관하는 타입 객체
```cpp

class ProjectileType{  
public:
	ProjectileType(const float dmg, float spd, Texture& tex)
	: damage(dmg),
	speed(spd),
	texture(tex){}

	float getDamage() const { return damage; }
	float getSpeed() const { return speed; }
	Texture& getTexture() const { return texture; }

	void render(float x, float y) const{
		cout << "Render [" << texture << "] at (" << x << ", " << y << ")"; 
	}

private:
	float damage;
	float speed;
	Texture texture;
};
```

> **타입 객체**
> 한 종류에 속하는 모든 인스턴스가 공통으로 가지는 속성을 담고 있는 객체

---

##### 3. 종류 별 인스턴스 생성 후 공유
```cpp
class ProjectileFactory{
public:
	static ProjectileType* Get(ProjectileKind kind){
		auto& pool = GetPool();
		auto it = pool.find(kind);
		if(it != pool.end())
			return it->second;

		//없으면 생성
		ProjectileType* type = nullptr;
		switch (kind){
			case ProjectileKind::Bullet:
				type = new ProjectileType("bullet.png",10.f,500.f);
				break;
			case ProjectileKind::Rocket:
				type = new ProjectileType("rocket.png",10.f,500.f);
				break;
		}
		pool[kind] = type;
		return type;
	}

private:
	static unordered_map<ProjectileKind, ProjectileType*>& GetPool(){
		unordered_map<ProjectileKind, ProjectileType*> pool;
		return pool;
	}
};
```
- ProjectileKind의 종류별로 한번만 ProjectileType 객체를 생성하고, 이후에는 같은 객체를 재사용하도록 한다.

---

##### 4. 개별 상태를 포함한 발사체 인스턴스
```cpp
struct Projectile{
	bool active = false;
	float x = 0; y = 0;
	float dirX = 0; dirY = 0;
	ProjectileType* type = nullptr;

	void init(ProjectileKind kind, float startX, float startY, float dx, float dy){
		type = ProjectileFactory::Get(kind);
		x = startX;
		y = startY;
		float len = sqrt(dx*dx + dy*dy);
		dirX = dx/len;
		dirY = dy/len;
		active = true;
	}
}
```
- 초기화 할때, 발사체 타입을 <span style="color:rgb(255, 192, 0)">공유 상태 클래스</span>로 설정한다.

> **struct를 사용한 이유**
> - 모든 멤버가 public이다.
> - 캡슐화가 필요 없고, 단순한 데이터로 사용할 것이기 때문

---

##### 5. Pooling을 이용하여 미리 생성 후 재사용
```cpp
class ProjectilePool{
public:
	// 미리 생성 (projectile 인스턴스가 메모리에 존재하게 됨)
	ProjectilePool(size_t capacity)
	: pool(capacity) {}

	//재사용
	void fire(ProjectileKind kind, float sx, float sy, float dx, float dy){
		for(auto& p : pool){
			if(!p.active){
				p.init(kind, sx,sy, dx, dy);
				return;
			}
		}
		//풀 부족 시 무시 또는 확장 로직 추가 가능
	}
private:
	vector<Projectile> pool;
};
```
- 개별 상태를 포함한 발사체 인스턴스 초기화

**재사용**
> - 매번 new Projectile을 하지 않고, 이미 만들어둔 pool안의 Projectile 객체 중 `active==false(빈 슬롯)`인 것을 골라 `init()`으로 위치, 방향, 타입만 새로 설정하여 **재사용**해준다.

**객체 풀링**
> - 미리 일정 개수 객체를 생성해 두고, 필요할 때 꺼내 쓰며, 사용이 끝나면 다시 pool에 되돌려 재활용하는 기법이다.
> - new/delete나 언리얼의 SpawnActor/DestroyActor 는 비용이 크다.
> - 메모리 사용량 예측이 가능하다.

---

##### 6. 사용 예시
```cpp
int main(){
	ProjectilePool pool(100);

	const float deltaTime = 1.f/60.f;
	for(int frame = 0; frame < 120; ++frame){
		if(frame % 10 == 0){
			pool.fire(ProjectileKind::Bullet, 400, 300, 1, 0);
			pool.fire(ProjectileKind::Rocket, 400, 300, 0, 1);
		}
	}
}
```
---