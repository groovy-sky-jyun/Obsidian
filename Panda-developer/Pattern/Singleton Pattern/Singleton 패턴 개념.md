### Singleton 패턴이란?
특정 클래스의 인스턴스가 프로그램 전체에서 단 하나만 존재하도록 보장하는 디자인 패턴이다.
- 오직 한 개의 클래스 인스턴스만 가진다.
- 전역 접근 

---

### 사용 예시
게임 내에서 여러 개의 인스턴스를 가지면 데이터가 꼬이거나 상태관리가 복잡해지는 기능들을 구현할 때 주로사용한다.
- `GameManager` : 게임 전체 상태 관리
- `SoundManager` : 모든 배경음악, 효과음 관리
- `PlayerDataManager` : 점수나 체력 데이터 저장

##### FileSystem 싱글톤 작성 예시시
```cpp title:FileSystem
class FileSystem{
public:
	static FileSystem& instance(){
		static FileSystem *instance = new FileSystem();
		return *instance;
	}

private:
	FileSystem();
}
```
- static을 통해 `FileSystem::instance()`가 처음 호출될 때 `instance` 값이 초기화 되고, 그 뒤로는 다시 호출되더라도 이미 생성된 고정 값 `instance`를 그대로 반환한다.
- private에 생성자를 둠으로써 다른 객체에서 new로 새로운 FileSystem 인스턴스를 생성하지 못한다.

---

### 장점
#### 1. 런타임에 초기화된다.
**정적 변수**는 주로 <span style="color:rgb(255, 192, 0)">컴파일</span> 타임에 초기화된다. (main 함수가 시작되기 전 값 할당됨)
그러면 컴파일 후에 알 수 있는 데이터를 초기화에 사용할 수 없다. 
또한, 정적 변수 초기화 순서도 컴파일러에서 보장해주지 않기에, 정적 변수끼리의 의존도 어렵다.

**싱글톤**은 <span style="color:rgb(255, 192, 0)">런타임</span> 중 인스턴스를 처음 호출하는 시점에 초기화된다.
때문에 다른 초기화된 데이터를 사용할 수 있다.
또한, 순환 의존만 없다면 초기화할 때, 다른 싱글톤을 참조할 수 있다.

#### 2. 싱글톤을 상속할 수 있다.
플랫폼 Will, PlayStaton 두개를 지원하는 게임을 개발한다고 해보자.
그럼 플랫폼이 뭐냐에 따라 동작이 달라져야 하는 기능들이 있을 것이다.
이것을 싱글톤으로 구현하고자 할 때, 상속을 통해 구현할 수 있다.
```cpp title:FileSystem hl:4,6,8,21,26
class FileSystem{ 
public:
	static FileSystem& instance(){
		#if PLATFORM == PLAYSTATION
		static FileSystem *instance = new PSFileSystem();
		#elif PLATFORM == WILL
		static FileSystem *instance = new WillFileSystem();
		#endif
		return *instance;
	}

	virtual ~FileSystem(){}
	virtual char* readFile(char* path) = 0;
	virtual void writeFile(char* path, char* contents) = 0;

protected:
	FileSystem(){}
};

//Platform 종류별 클래스
class PSFileSystem : public FileSystem{
public:
	virtual char* readFile(char* path){}
	virtual void writeFile(char* path, char* contents){}
};
class WillFileSystem : public FileSystem{
public:
	virtual char* readFile(char* path){}
	virtual void writeFile(char* path, char* contents){}
};
```
- 싱글톤 상속을 통해 간단하게 플레이어 플랫폼에 맞는 인스턴스를 생성할 수 있다.
- 즉, 런타임에 단일 인스턴스의 구체적인 구현을 유연하게 변경할 수 있다.
