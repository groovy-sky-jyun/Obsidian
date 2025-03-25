### 벡터
array는 정적 배열로 크기를 자유롭게 변경하지 못한다.
vector는 이러한 단점을 보완하여 크기를 동적으로 조정할 수 있다.
또한 인덱스를 통한 접근으로 빠른 속도를 가진다. O(1)

### 벡터 초기화
```cpp
#include <iostream>
#include <vector>
using namespace std;

int main(){
	vector<int> v;
	vector<int> v(6); //크기 6 할당 -> 요소 추가 시 크기 자동 증가
	vector<int> v[3]; //크기가 3인 배열(원소의 크기는 동적)
	vector<vector<int>> v; //2차원 벡터
}
```

#### 이차원 배열 벡터
벡터를 2차원 동적 배열로 사용할 수 있다.
```cpp title:vector_array, hl:6,10 
#include <iostream>
#include <vector>  
using namespace std;

int main(){
	vector<int> vec[3]; //크기가 3인 배열
	vec[0] = {1, 2, 3, 4, 5, 6}; //원소의 크기는 동적 작용
	vec[3] = {4, 5, 6}; //vec의 크기를 벗어난 접근은 오류 발생
	
	vector<vector<int>> v; //2차원 벡터
}
```
##### 6line
`vector<int> v[3];` 의 경우 <mark style="background: #FFB8EBA6;">행의 크기가 3으로 고정</mark>이지만, <mark style="background: #FFB8EBA6;">열의 크기는 동적</mark>으로 사용할 수 있다. 
대신, 행의 크기가 3이 넘은 곳을 접근하려 하면 배열의 범위를 벗어난 접근이므로 오류가 발생한다.

##### 10line
`vector<vector<int>> v;`의 경우 <mark style="background: #FFB8EBA6;">행과 열의 크기를 동적</mark>으로 사용할 수 있는 이차원 배열 벡터이다. 메모리가 동적으로 할당되므로 push_back()으로 유연한 추가가 가능하다.

---
### 벡터의 Iterators(반복자)
컨테이너 내부에서 동작하는 객체이다.
포인터는 단순한 메모리 주소지만, 반복자는 컨테이너의 원소를 안전하게 순회하는 인터페이스를 제공한다.
##### 사용 방법
`vector<int>::iterator itr;`
```cpp
#include <iostream>
#include <vector>
using namespace std;
int main(){
	vector<int> v = {1,2,3,4,5};
	vector<int>::iterator itr = v.begin();
	
	for(; itr != v.end(); itr++){
		cout << *itr << '\n'; //출력: 1 2 3 4 5
	}
}
```

>##### #begin
`v.begin()` 은 벡터 <span style="color:rgb(255, 192, 0)">시작점</span>의 주소 값을 반환한다.
>
>##### #end 
>`v.end()` 은 벡터 <span style="color:rgb(255, 192, 0)">끝부분 + 1</span> 의 주소 값을 반환한다.
>그렇기 때문에 `*v.end()` 으로 마지막 요소 값을 사용하려 하면 오류가 발생한다.
>마지막 요소의 값을 사용하기 위해서는 `*(--v.end())` 를 사용하거나 `v.back()`을 사용해야 한다.
>
>![[Pasted image 20250321143352.png|https://en.cppreference.com/w/cpp/container/vector/rend]]

<br>

### 벡터 요소 접근

__ 1) <span style="color:rgb(188, 149, 218)">v.at(i)</span>__
- 벡터의 i번째 인덱스 요소 값
- 범위 체크를 하므로 안전성이 보장되지만 그로 인해 속도가 느릴 수 있다.

__2) <span style="color:rgb(188, 149, 218)">v[i]</span>__
- 벡터의 i번째 인덱스 요소 값
- 범위 체크를 하지 않기에 안정성이 보장되지 않지만 속도가 빠르다.
- 벡터는 효율을 중점으로 둔 라이브러리 함수이기에 보통v[ ]를 권장한다.

__3) <span style="color:rgb(188, 149, 218)">v.front()</span>__
- 벡터의 첫번째 요소 값

__4) <span style="color:rgb(188, 149, 218)">v.back()</span>__
- 벡터의 마지막 요소 값

---

### 벡터 요소 삽입


__1) <span style="color:rgb(188, 149, 218)">v.insert(iterator pos, type val)</span>__
- pos 위치에 val 삽입
- 임시객체를 복사하여 vector에 추가한다. (복사/이동 발생)
- 이미 생성된 객체를 삽입할 때 사용
```cpp
#include <iostream>
#include <vector>
using namespace std;

class Cat {
public: 
    Cat(int val) { value = val; }
    int value;
};
int main() {
    vector<Cat> v;
    Cat cat1(1);

    v.insert(v.end(), cat1); //기존 객체 복사
    cout << v[0].value << '\n'; //출력: 1

    cat1.value = 5;
    cout << v[0].value << '\n'; //출력: 1
}
```
cat1은 이미 생성된 객체이므로 emplace를 사용하는것 보다는 insert를 사용하는 것이 더 적합하다. 
이때, 객체가 복사되어 벡터에 저장되는 것이기 때문에 cat1.value의 값을 5로 변경해주어도 벡터의 v[0].value는(복사된 객체이므로) 영향을 받지 않는다.

__2) <span style="color:rgb(188, 149, 218)">v.emplace(iterator pos, value_type& val)</span>__
- 원하는 위치에 요소 삽입
- 컨테이터 내부에서 객체를 생성하므로 복사 x
- 새로운 객체를 생성하고자 할 때 사용(성능 측면 더 우수)
```cpp
#include <iostream>
#include <vector>

using namespace std;

class Cat {
public: 
    Cat() { value = 0; }
    Cat(int val) { value = val; }
    int value;
};
int main() {
    vector<Cat> v;
    
    v.emplace(v.end(), 2); //Cat(2) 생성자 호출 -> 새로운 객체 생성 
    v.emplace(v.end()); //Cat() 생성자 호출 -> 새로운 객체 생성

    cout << v[0].value << '\n'; //출력: 2
    cout << v[1].value << '\n'; //출력: 0
}
```
새로운 객체를 생성하고자 할때는 생성자를 호출한다고 생각하면 된다. 즉, 생성자의 매개변수값을 val로 넘겨주면 되는 것이다.

__3) <span style="color:rgb(188, 149, 218)">v.push_back(type val)</span>__
-  insert와 같이 이미 생성된 객체를 추가할 때 사용한다.
- 다른점은 요소를 추가하는 위치가 벡터의 제일 마지막으로 고정되어 있는 것이다.
- 복사 o

__4) <span style="color:rgb(188, 149, 218)">v.emplace_back(type val)</span>__
- emplace와 같이 새로운 객체를 추가할 때 사용한다.
- 다른점은 요소를 추가하는 위치가 벡터의 제일 마지막으로 고정되어 있는 것이다.
- 복사 x

---

### 벡터 요소 제거
__1) <span style="color:rgb(188, 149, 218)">v.pop_back()</span>__
- 벡터의 마지막 요소 제거 (반환값x)

__2) <span style="color:rgb(188, 149, 218)">v.erase(iterator pos)</span>__
- 삭제할 요소의 특정 위치를 받아 제거한다. (단일 요소 제거)
- 제거된 요소의 <mark style="background: #FFB8EBA6;">다음 요소 위치를 가리키는 iterator를 반환</mark>한다.
- 삭제된 요소 뒤의 모든 요소들이 앞으로 이동해야 하므로 가장 앞의 요소를 삭제 하게 되면 O(N)의 시간이 걸린다.
- 가장 마지막 요소를 제거하려면 `v.erase(v.end()-1);`을 해주어야 한다.

__3) <span style="color:rgb(188, 149, 218)">v.erase(iterator first, iterator last)</span>__
- 특정 범위인 <span style="color:rgb(255, 192, 0)">first</span> ~ (<span style="color:rgb(255, 192, 0)">last - 1</span>) 범위의 요소들을 제거한다.
- <mark style="background: #FFB8EBA6;">끝 위치(last)를 가리키는 iterator를 반환</mark>한다.
``` cpp
#include <iostream>
#include <vector>

using namespace std;

int main() {
    vector<int> v = {1, 2, 3, 4, 5};
    vector<int>::iterator itr;
    itr = v.erase(v.begin() + 1, v.begin() + 4); //2, 3, 4 삭제
	cout << *itr << '\n'; //출력: 5

}
```

__4) <span style="color:rgb(188, 149, 218)">v.clear()</span>__
- 벡터의 모든 요소를 지운다. `vector.size()` -> 0

---

### 정렬
원소들을 오름차순으로 정렬할 수 있다.
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;
int main(){
	vector<int> v = {1, 2, 3};
	sort(v.begin(), v.end());
}
```

rbegin()
은 reverse_iterator 역방향 iterator를 반환한다.
```cpp
vector<int> v = { 1, 2, 3 };

vector<int>::reverse_iterator itr = v.rbegin();
for (; itr != v.rend(); itr++) {
    cout << *itr << '\n'; // 출력: 3 2 1
}
```
역방향의 시작 요소를 가리키고 있다. 하지만 ++를 해줘야 내림차순이 된다.
--를 하면 오류 발생한다.