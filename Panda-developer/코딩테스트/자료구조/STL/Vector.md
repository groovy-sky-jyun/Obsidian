### 벡터 함수
#### emplace(iterator pos, 삽입 값)

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main(){
	std::vector<int> numbers = {1,2,4,5};
	numbers.emplace(numbers.begin() + 2, 3);

	for(const int &num : numbers){
		cout << num << "\n";
	} //출력: 12345	
}
```
#iterator 는 #반복자 이다.
<mark style="background: #FFF3A3A6;">number.begin()</mark>은 첫번째 원소를 가리키는 <mark style="background: #FFF3A3A6;">반복자</mark> 이다.
>##### #반복자
>컨테이너 내부에서 동작하는 객체이다.
>포인터는 단순한 메모리 주소지만, 반복자는 컨테이너의 원소를 안전하게 순회하는 인터페이스를 제공한다.
#### erase