### string 클래스
string은 c++ 표준 라이브러리에서 제공하는 클래스이다.
string 클래스를 사용하기 위해선 아래의 코드가 선언되어야 한다.
```cpp
#include<string>
using namespace std;
```

---

### append(string& str)
문자열 끝에 문자열을 붙일 수 있다.
```cpp
#include<iostream>
#include<string>
using namespace std;
int main(){
	string s = "hello ";
	s.append("how are you?");
	cout << s << '\n'; //출력: hello how are you?
}
```

### insert(int pos, string &str)
문자열 사이에 새로운 문자를 집어넣을 수 있다.
pos 위치에 str 문자열 삽입
```cpp
#include<iostream>
#include<string>
using namespace std;
int main(){
	string s = "paa";
	s.insert(2,"nd");
	cout << s << '\n'; //출력: hello how are you?
}
```

### replace(int pos, int n, string& str)
pos위치부터, n개 문자를, str문자열로 대치
```cpp
#include<iostream>
#include<string>

using namespace std;
int main() {
	string s = "panda";
	s.replace(1, 2, "aaaaa");
	cout << s << '\n'; //출력: paaaaada
}
```

---

### erase(int pos, int n)
pos 위치부터 n개 문자 삭제 (삭제된 만큼 뒤에 문자들이 앞당겨진다.)
-> 배열의 크기가 줄어든다.
```cpp
#include<iostream>
#include<string>

using namespace std;
int main() {
	string s = "panda";
	cout << s.size() << '\n'; //출력: 5

	s.erase(0, 2);
	cout << s.size() << '\n'; //출력: 3
}
```

---

### length() / size()
~~`#include<string>`~~
String의 길이를 반환
`"Hello".length()` -> 5

---

### at(int pos)
pos 위치의 문자 리턴
`string s = "abc"; cout << s.at(1);` -> 출력: b

### substr(int pos, int n)
- string의 pos위치부터 n개 문자를 새로운 서브 스트링으로 리턴
`"Hello".substr(1,3)` -> "el"

---

### toupper()/tolower()
대문자로 변환/소문자로 변환
주의할 점은 문자열을 변환해 주는 것이 아니라 'char 문자' 타입만 변환해준다. 또한, 반환된 값은 ASCII 코드값으로 <span style="color:rgb(255, 192, 0)">int형으로 반환</span>한다. 
```cpp
#include<iostream>
#include<string>

using namespace std;
int main() {
	string s = "abcd";
	cout << (char)toupper(s[0]) << '\n'; 
	//대문자 A ASCII 코드 값: 65
	//char 형변환으로 출력: A

	cout << tolower(s[0]) << '\n';
	//소문자 a ASCII 코드 값: 95
	// 출력: 95

	cout << toupper(s) << '\n';
	//문자열은 오류
}
```

---

### find(string) / find(string, int)
- 문자열의 처음부터 str 검색하여 발견한 처음 인덱스 리턴
- 없으면 -1 리턴
- /문자열의 int 위치부터 str 발견한 처음 인덱스 리턴
- 없으면 -1 리턴
`"Hello".find("l")` -> 2
`"Hello".find("k")` -> -1

### compare(string &str)
- 두 문자열의 ASCII 순서 비교
- <span style="color:rgb(255, 192, 0)">char(문자) 끼리 비교하면 컴파일 에러</span>
- 앞이 크면 - (음수)
- 뒤가 크면 + (양수)
- 같으면 0
`"a".compare("b")` -> -
`"b".compare("a")` -> +
`"a".compare("a")` -> 0

