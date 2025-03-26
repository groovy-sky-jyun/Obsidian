enum class의 항목에는 값을 명시하지 않으면 이전 값에서 자동 1 증가한다.

아무런 항목에도 값을 명시해주지 않으면 0부터 시작된다.

아래의 예시를 보면 A를 1로 지정해줬으므로 B=2, C=3, D=4, E=5가 된다.

A를 3으로 지정해준다면 B는 4가 되는 것이다.

```cpp
UENUM()
enum class ENumber : uint8
{
	A = 1,
	B,  // 2
	C,  // 3
    	D,  // 4
    	E   // 5
}
```

enum의 항목 값은 기본으로 정수형을 가지기에 문자열이나 숫자가 아닌 다른 값을 할당할 수는 없다.

```cpp
UENUM()
enum class ENumber : uint8
{
	A = "One",  // 오류: 문자열은 잘못된 값
	B,
	C, 
    	D, 
    	E
}
```