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
> - <span style="color:rgb(255, 192, 0)">미리 일정 개수 객체를 생성</span>해 두고, 필요할 때 꺼내 쓰며, <span style="color:rgb(255, 192, 0)">사용이 끝나면 다시 pool에 되돌려 재활용</span>하는 기법이다.
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

### 발생할 수 있는 문제점
#### 1. 동시성 문제 (Flyweight + Pooling)
```cpp hl:4,8
static ProjectileType* Get(ProjectileKind kind){
	auto& pool = GetPool();
	   
	auto it = pool.find(kind);
	if(it != pool.end())
		return it->second;

	ProjectileType* type = new ProjectileType();
	pool[kind] = type;
	return type;
	}
```
**문제 상황**
A, B 스레드(<span style="color:rgb(255, 192, 0)">다중 스레드</span>)가 거의 동시에 `pool.find(kind)` 호출 후, pool에 kind가 없다고 판단해, A, B 모두 `new ProjectileType()`호출 

**문제 발생**
<span style="color:rgb(255, 192, 0)">pool의 중복 생성</span>으로 덮어쓰기, 메모리 누수, 포인터 충돌 등의 예상치 못한 버그 발생

**해결책**
<span style="color:rgb(255, 192, 0)">뮤텍스</span>로 임계 영역 보호
 ```cpp
 static mutex mtx;
 lock_guard<mutex> lock(mtx);
 ```

<br>

#### 2. 간접 참조로 인한 성능 오버헤드
```cpp hl:3
void updateAll(float dt){
	for(auto& p : projectiles){
		float speed = p.type->getSpeed();
		p.x += p.dirX * speed * dt;
	} 
}
```

**문제 상황**
수천 번 호출되는 Hot Loop에서 `포인터 -> 멤버`로의 두 단계 메모리 접근

**문제 발생**
포인터가 가리키는 주소가 L1캐시에 없을 경우 L2 -> L3 -> 메인 메모리(DRAM) 순으로 조회해야 한다. 이런 <span style="color:rgb(255, 192, 0)">캐시 미스</span>로 인한 누적 지연이 프레임 드롭으로 이어질 수 있다. 

**해결 방법**
값이 자주 사용돼서 캐시 히트가 중요한 값일 경우, 개별 상태에 정의하여 <span style="color:rgb(255, 192, 0)">포인터 체이싱을 제거</span>한다.
```cpp hl:4,14
struct Projectile{
	float speed = 0.f;
	float damage = 0.f;
	void init(ProjectileKind k, ...){
		auto* t = ProjectileFactory::Get(k);
		speed = t->getSpeed();
		damage = t->getDamage();
		active = true;
	}
}; 

void updateAll(float dt){
	for(auto& p : projectiles){
		p.x += p.dirX * p.speed * dt;
	}
}
```
- hl:4 에서 <span style="color:rgb(255, 192, 0)">초기화할 때 한번만 포인터에 접근</span>하도록 한다.
- hl:14 에서 <span style="color:rgb(255, 192, 0)">Hot Path에서 포인터 간접 참조를 제거</span>한다.

>**공유 상태는 어떤 값들을 모아두면 좋을 까?**
>- 용량이 큰 자원: 텍스쳐, 메시, 사운드 등
>- 변하지 않는 설정 값(읽기 전용) -> 인스턴스 간 중복 제거
>- 드물게 참조되는 속성
>- 대용량 상수 데이터 : 파티클 프리셋, 물리-머티리얼 파라미터, 커브 등등

<br>

#### 3. 수명 관리와 메모리 누수 위험
```cpp hl:5
class ProjectileFactory{
public: 
	static ProjectileType* Get(ProjectileKind k){
		...
		ProjectileType* t = new ProjectileType("bullet.png", 10.f, 500.f);
		pool[k] = t;
		return t;
	}
};
```
**문제 상황**
객체를 static 변수나 싱글턴 인스턴스에 보관해 두고, 프로그램이 끝날때 <span style="color:rgb(255, 192, 0)">메모리 해제를 해주지 않은 경우</span>

**문제 발생**
new로 할당한 객체가 해제되지 않아 <span style="color:rgb(255, 192, 0)">메모리 누수 발생</span>

**해결 방법**
<span style="color:rgb(255, 192, 0)">스마트 포인터</span>를 사용하여 종료 시 자동으로 delete가 되도록 해준다.
```cpp
class ProjectileFactory{
	...
private:
	static vector<unique_ptr<ProjectileType>>& GetPool(){
		static vector<unique_ptr<ProjectileType>> pool(static_cast<size_t>(ProjectileKind::Count));
		return pool;
	}
};
```
