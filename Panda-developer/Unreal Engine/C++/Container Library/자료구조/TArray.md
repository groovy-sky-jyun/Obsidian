## TArray 

TArray는 동적 배열인 컨테이너 라이브러리이다.

엘리먼트가 순서를 가진다. 이를 인덱스로 접근할 수 있다.

TArray를 new, delete로 생성과 소멸을 하는것은 좋지 않다.

즉, 데이터가 많을 수록 검색, 삭제, 수정 작업은 비용이 많이 발생한다. 하여 해당 작업들을 빈번하게 사용한다면 TSet을 사용하는것이 좋다.

```cpp
TArray<int32> IntArray;
```

#### Init

특정값을 원하는 개수만큼 추가할 수 있다.

반복된 값을 배열에 채우고자 할 때 사용한다.

```cpp
IntArray.Init(10, 5);
// IntArray == [10,10,10,10,10]
```

#### Add와 Emplace의 차이

배열에 엘리먼트를 추가할 때, **Add**는 값을 복사해서 배열에 추가하는 방식을 적용한다.

**Emplace**는 배열에 새로운 객체를 생성한 뒤 그 객체의 값을 초기화 시켜주는 방식을 적용한다.

이때의 큰 차이점이 복사를 하느냐 하지않느냐 이다. 

Emplace는 복사를 하는 대신 객체를 초기화시켜줌으로서 복사비용을 줄일 수 있다.

그렇기에 추가할 엘리먼트를가 크고 복잡하다면 Add대신 Emplace를 사용하는 것을 추천한다.

> **Emplace에서도 객체를 초기화할 때 기존 값을 복사해서 넣어야하지않나?**  
> **그럼 결국에는 둘다 복사비용이 드는거 아닌가?**  
>   
> 아니다. Emplace에서 객체를 초기화 할때는 기존값을 복사하여 초기화하는것이 아니라 전달받은 인수들로 초기화를 하는것이다. 그렇기에 Add와 똑같은 복사과정을 거친다고 보기 어렵다.

#### Append

다수의 엘리먼트를 한번에 추가할 수 있다.

```cpp
FString Arr[] = { TEXT("of"), TEXT("Tomorrow") };
StrArr.Append(Arr, ARRAY_COUNT(Arr));
// StrArr == ["Hello","World","of","Tomorrow"]
```

#### AddUnique

기존 배열에 추가하려는 엘리먼트가 없는 경우에만 추가한다.

이때 존재여부는 operator를 사용해서 검사하게 된다.

```cpp
StrArr.AddUnique(TEXT("!"));
// StrArr == ["Hello","World","of","Tomorrow","!"]
 
StrArr.AddUnique(TEXT("!"));
// StrArr is unchanged as "!" is already an element
```

#### Insert

추가하고자하는 인덱스 위치에 엘리먼트를 추가할 수 있다.

맨 끝에 element를 추가하는 것은 가벼우나, 중간에 요소를 추가하거나 삭제하는 작업은 비용이 크다.

Insert도 마찬가지로 전체적인 메모리를 변경해 주어야 하기때문에 비용이 많이 발생한다.

```cpp
// StrArr == ["Hello","World","of","Tomorrow","!"]
StrArr.Insert(TEXT("Brave"), 1);
// StrArr == ["Hello","Brave","World","of","Tomorrow","!"]
```

#### Contains와 Find의 차이

**Contains**는 단순히 배열에 해당 엘리먼트가 있는지 확인하고 결과값을 bool로 반환한다.

**Find**는 배열에 엘리먼트가 있는지 확인하고 있다면 해당 엘리먼트를 반환하고 없다면 INDEX_NONE을 반환하게 된다.

원하는 엘리먼트가 있는지 확인 후 그 값을 변수로 사용하고 싶다면 Find를 쓰면 된다.

이러한 검색의 경우 모든 요소를 순회해야할 수 있기 때문에 비용이 많이 들 수 있다. 때문에 검색기능을 빈번하게 사용해야한다면 TSet을 이용하는것을 추천한다.

#### RemoveAt

원하는 인덱스의 값을 삭제할 수 있다. 이때는 배열의 크기를 확인해줘야한다. 배열의 범위를 벗어난 인덱스를 삭제하려 하면 런타임 오류가 발생한다.

element를 삭제하고나면 그 뒤의 element들이 앞인덱스로 당겨지기 때문에 배열에는 빈 공간이 절대 생기지 않는다.