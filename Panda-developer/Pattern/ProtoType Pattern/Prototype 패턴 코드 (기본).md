### 몬스터 복제 
#### Monster
```cpp title:Monster 
class Monster{
public:
	virtual ~Monster() {}
}
```

---

#### Ghost
```cpp title:Ghost
class Ghost : public Monster {
public:
	Ghost() 
	: Ghost(100, 3){}
	
	Ghost(int health, int speed)
	: health_(health),
	  speed_(speed){}

private:
	int health_;
	int speed_;
}
```

---

#### Spawner
```cpp title:Spawner
typedef Monster* (*SpawnCallback)();

class Spawner{
public:
	explicit Spawner(SpawnCallback spawn) 
	: spawn_(spawn){}

	Monster* spawnMonster(){
		return spawn_(); //실제로 함수 호출
	}

private:
	SpawnCallback spawn_;
}

Monster* spawnGhost(){
	return new Ghost();
}
```

```cpp
int Main(){
	Spawner* ghostSpawner = new Spawner(spawnGhost);
	Monster* ghost = ghostSpawner->spawnMonster();
}
```

**Callback**
함수 자체 주소를 넘겨준다.
위에서는 Monster* 를 반환하는 함수의 주소를 담고 있는 함수 포인터 타입의 별칭을 SpawnCallback 라고 정의한 것이다.

explicit Spawner(SpawnCallback spawn) 여기에서 spawn은 SpawnCallback 타입의 매개변수이다.

(explicit는 암시적 변환을 금지하기 위함이다.)

Spawner* ghostSpawner = new Spawner(spawnGhost); 에서 spawnGhost()가 아닌 spawnGhost 를 매개변수로 넘겨주는이유?
- spawnGhost는 함수 포인터를 전달하는 것이고, spawnGhost()는 함수 호출 결과를 전달하는 것이기에 후자는 타입이 맞지않아 오류가 발생한다.

---

```cpp title:Monster 
class Monster{
public:
	virtual ~Monster() {}
	virtual Monster* clone() const = 0;
};
```


#### Ghost
```cpp title:Ghost
class Ghost : public Monster {
public:
	Ghost() 
	: Ghost(100, 3){}
	
	Ghost(int health, int speed)
	: health_(health),
	  speed_(speed){}

	vortual Monster* clone(){
		return new Ghost(helath_,speed_);
	}

private:
	int health_;
	int speed_;
};
```



#### Spawner
```cpp title:Spawner
class Spawner{
public:
	Spawner(Monster* prototype) 
	: prototype_(prototype){}

	Monster* spawnMonster() const{
		return prototype_->clone();
	}

private:
	const Monster* prototype_;
};
```

```cpp
int Main(){
	Monster* ghostPrototype = new Ghost(15,3);
	Spawner* ghostSpawner = new Spawner(ghostPrototype);
};
```


---


```cpp title:Monster 
class Monster{
public:
	virtual ~Monster() {};
};
```

#### Ghost
```cpp title:Ghost
class Ghost : public Monster {
public:
	Ghost() 
	: Ghost(100, 3){}
	
	Ghost(int health, int speed)
	: health_(health),
	  speed_(speed){}

	Monster* clone(){
		return new Ghost(helath_,speed_);
	}

private:
	int health_;
	int speed_;
};
```


#### Spawner
```cpp title:Spawner
class Spawner{
public:
	virtual ~Spawner(){}
	virtual Monster* spawnMonster() = 0;
};

template <class T>
class SpawnerFor : public Spawner {
public:
	virtual Monster* spawnMonster() override {
		return new T(); 
	}
};
```

```cpp
int Main(){
	Spawner* ghostSpawner = new SpawnerFor<Ghost>();
	Monster* ghost = ghostSpawner->spawnMonster();
};
```