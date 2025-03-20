### equals
두 String이 같은지 판단
`"Hello".equals("Hello")` -> false

### length
String의 길이를 반환
`"Hello".length()` -> 5

### toUpperCase
대문자로 변환
`"Hello".toUpperCase` -> "HELLO"

### toLowerCase
소문자로 변환
`"Hello".toLowerCase` -> "hello"

### find / contains
- `#include<string>`을 추가해야 한다.
- 문자열에 해당하는 문자열이 있으면 인덱스 위치 반환<span style="color:rgb(255, 192, 0)">(0부터)</span>
- 여러개가 해당한다면 제일 첫 해당하는 위치 반환
- 없으면 string::npos(== 쓰레기 값) 반환
`"Hello".find("l")` -> 2
`"Hello".find("k")` -> string::npos

### substring / at
- string의 일부를 추출
- 시작 인덱스와 끝 인덱스를 포함하여 자른다.
`"Hello".substring(1,3)` -> "el"
> ##### at (문자 추출)
> `"Hello".at(1)` -> "e"

### replace
1. 문자열에 해당하는 문자열이 있는지 찾는다.
2. 있으면 해당 문자와 매개변수값을 대체
3. `"Hello".replace("l"," ")` -> "He  o" 

### trim()
앞뒤 공백 제거
- #include<string> 필요
- 문자열 가운데에 있는 공백들은 제거하지 않는다.
```cpp
#include<string>
int main(){
	string a = "   Hello my name is panda ! "
	cout << trim(a) << '\n'; 
	//출력 : "Hello my name is panda !"
}
```

### compare
- 두 문자열의 ASCII 순서 비교
- 앞이 크면 -
- 뒤가 크면 +
- 같으면 0
`"a".compare("b")` -> -
`"b".compare("a")` -> +
`"a".compare("a")` -> 0

### remove vs erase