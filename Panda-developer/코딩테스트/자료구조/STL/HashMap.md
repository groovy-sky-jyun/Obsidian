key-value 쌍으로 데이터를 저장하는 자료구조이다.
키를 통해 값을 빠르게 조회할 수 있다.
내부적으로 해시 테이블을 사용하여 데이터를 저장한다.
- 삽입 순서 상관없이 저장되어 랜덤하게 보인다.
- 키에 따른 값들은 버킷에 저장된다.(랜덤위치)
- `#include <unordered_map>` 을 사용하여 해시맵을 구현할 수 있다.
>#### 중복 키
>해시맵은 중복 키를 허용하지 않는다. 기존에 키가 있으면 해당 값에 덮어씌워지게 된다.


### 선언
```cpp
#include <iostream>
#include <unordered_map>
using namespace std;

int main(){
	unordered_map<string, int> hashmap; //<key, value>
}
```

--- 

### 값 삽입
__1) <span style="color:rgb(188, 149, 218)">insert({key, value})</span>__
- ` hashmap.insert({ "apple", 1 });`
- 중복된 키를 입력할 경우 삽입 무시(덮어씌워지기 x)
- 중복 삽입 x
```cpp
int main() {
    unordered_map<string, int>hashmap;

    hashmap.insert({ "apple", 1 });
    hashmap.insert({ "apple", 3 }); //중복 삽입 x
    
    cout << hashmap["apple"] << '\n'; //출력: 1
```

__2) <span style="color:rgb(188, 149, 218)">h[key] = value</span>__
- ` hashmap["banana"] = 2;`
-  중복된 키를 입력할 경우 값이 덮어 씌워짐
-  중복 삽입 o
```cpp
int main() {
    unordered_map<string, int>hashmap;

    hashmap.insert({ "apple", 1 });
    hashmap["apple"]=3; //중복 삽입 o
    
    cout << hashmap["apple"] << '\n'; //출력: 3
```

---

### 값 찾기
__<span style="color:rgb(188, 149, 218)">find(key)</span>__
- key값이 hashmap에 존재하면 해당 위치를 가리키는 iterator를 반환한다.
- 없다면 end() iterator를 반환한다.
- 반환값으로 key, value를 접근할 수 있다. 
- key는 <span style="color:rgb(255, 192, 0)">first</span>, value는 <span style="color:rgb(255, 192, 0)">second</span>
```cpp
int main(){
	unordered_map<string, int>hashmap;

	hashmap["apple"] = 1;
	hashmap["banana"] = 2;

	cout << hashmap.find("apple")->first << '\n'; //key 반환
	cout << hashmap.find("apple")->second << '\n'; //value 반환	
}
```

__<span style="color:rgb(188, 149, 218)">count(key)</span>__
- 특정 키가 존재하는지 확인할 때 사용
- 키가 존재하면 1 반환
- 없으면 0 반환
```cpp
h["apple"] = 1;

if (h.count("applee")) { //0 반환
    cout << "success";
}
cout << "fail"; //출력: fail
```
---

### 값 접근
__<span style="color:rgb(188, 149, 218)">find(key) : iterator->first/second</span>__
```cpp
int main() {
    unordered_map<string, int>h;

    h["apple"] = 1;
    h["banana"] = 2;
    h["orange"] = 3;

    unordered_map<string, int>::iterator itr;
    for (itr = h.find("apple"); itr != h.end(); itr++) {
        cout << itr->first << '\n';
        cout << itr->second << '\n';
    }
}
```

__<span style="color:rgb(188, 149, 218)">begin()와 end()</span>__
```cpp
unordered_map<string, int>::iterator itr;
for (itr = h.begin(); itr != h.end(); itr++) {
    cout << itr->first << '\n';
    cout << itr->second << '\n';
}
```

__<span style="color:rgb(188, 149, 218)">hashmap[key]</span>__
```cpp
h["apple"] = 1;
h["banana"] = 2;
h["orange"] = 3;

cout << h["apple"] << '\n'; //출력: 1
cout << h["banana"] << '\n'; //출력: 2
```

---

### 삭제
__<span style="color:rgb(188, 149, 218)">erase(key)</span>__
- 아무것도 반환하지 않는다.

__<span style="color:rgb(188, 149, 218)">erase(iterator)</span>__
- 삭제한 요소의 다음 요소를 가리키는 iterator를 반환한다.
- 삭제된 요소가 마지막 요소라면 end() iterator를 반환한다.

__<span style="color:rgb(188, 149, 218)">clear()</span>__
- 모든 원소를 삭제한다. (반환값x)

---

### 그 외
<span style="color:rgb(188, 149, 218)">size()</span>
<span style="color:rgb(188, 149, 218)">empty()</span> 