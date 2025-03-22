`#include <queue>`
push(val)
pop()
front()
back()
empty()
size()

### priority_queue
완전 이진 트리(힙) 구조로 삽입
```cpp
struct Person {
    string name;
    int age;

    // 나이가 적을수록 우선순위가 높은 설정 (최소 힙)
    bool operator>(const Person& other) const {
        return age > other.age; // 나이가 많을수록 우선순위가 낮음
    }
};

int main() {
    // 최소 힙으로 설정: 나이가 적을수록 우선순위가 높음
    priority_queue<Person, vector<Person>> pq;
```