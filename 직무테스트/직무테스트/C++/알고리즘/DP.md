매개변수값을 작게 쪼개면서 재귀함수를 실행하는 방법
피보나치 수열
```cpp
#include <iostream>
using namespace std;

int fib(int n) {
    if (n <= 1) return n;
    return fib(n - 1) + fib(n - 2); // 중복 계산 발생!
}

int main() {
    cout << fib(10) << endl; // 55
    return 0;
}
```