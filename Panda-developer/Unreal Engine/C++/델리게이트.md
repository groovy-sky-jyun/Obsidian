### 델리게이트

"델리게이트를 통해 함수를 동적으로 호출할 수 있다."

특정 객체의 멤버 함수를 바인딩할 수 있고, 특정 이벤트가 발생했을 때 그 함수가 호출되도록 설정할 수 있다.

(함수 호출을 코드에서 지정하지 않고, 이벤트 발생시 해당 함수가 자동으로 실행되도록 설정할 수 있다.)

다른 객체의 함수를 호출할 때, 다른 객체의 유형을 모르더라도 해당 함수에 접근하고 실행할 수 있는 이점이 있다.

이를 통해 객체 간의 결합도를 낮추고, 코드의 유연성 및 재사용성을 높일 수 있다.

아래 예시로 우선 델리게이트가 어떤건지 알아보자.

---

#### 델리게이트 멀티 캐스트 예시)

아래의 그림은 플레이어의 체력이 변할 때 마다 Widget의 체력바 UI를 변경하고, Monster의 공격력에 변화를 주어야 하는 상황을 가정한 코드이다. 즉 2개의 외부 클래스 함수를 호출해야하는 상황이다.

델리게이트를 사용하지 않으면, Character 클래스에서는 Widget 클래스와 Monster 클래스를 include 한 뒤에 해당 클래스에 어떤함수가 있는지 확인하고 호출해야한다.

하지만 이런 방식은 결합도를 높이고 코드의 유연성을 해친다.

Character 클래스에서 Widget 클래스와 Monster 클래스를 몰라도 되게 하고 싶은것이다.

그래서 델리게이트를 사용한다. 델리게이트를 사용한 방법은 아래와 같다.

Widget 클래스에서는 OnHealthChanged 라는 델리게이트에 UpdateHealthBar() 함수를 바인딩 하고,

Monster 클래스에서는 OnHealthChanged 라는 델리게이트에 UpdateDamageLevel() 함수를 바인딩 하였다.

Character 클래스는 델리게이트를 정의하고, 호출하는 역할을 한다.

Character 클래스에서 델리게이트를 호출하면 OnHealthChanged 델리게이트에 바인딩 된 UWidget 클래스의 UpdateHealthBar() 함수와, Monster 클래스의 UpdateDamageLevel() 함수가 자동으로 실행된다.

이때 순서는 먼저 바인딩된 순차적으로 거의 동시에 실행된다고 보면 된다.

![[Pasted image 20250306230715.png|델리게이트 예시]]

---

#### Single - Cast 델리게이트 선언

**1) 반환값이 없는 경우**

- DECLARE_DELEGATE(FDelegateName)
- DECLARE_DELEGATE_OneParam(FDelegateName, Param1Type)
- ....
- DECLARE_DELEGATE_NineParams(FDelegateName, Param1Type, ..., Param9Type)

**2) 반환값이 없고, BP에서 사용할 경우**

- DECLARE_DYNAMIC_DELEGATE(FDelegateName)
- DECLARE_DYNAMIC_DELEGATE_OneParam(FDelegateName, Param1Type)
- ....
- DECLARE_DELEGATE_DELEGATE_NineParams(FDelegateName, Param1Type, ..., Param9Type)

**3) 반환값이 있는 경우**

- DECLARE_DELEGATE_RetVal(RetValType, FDelegateName)
- DECLARE_DELEGATE_RetVal_OneParam(RetValType, FDelegateName, Param1Type)
- ....
- DECLARE_DELEGATE_RetVal_NineParams(RetValType, FDelegateName, Param1Type, ..., Param9Type)

**4) 반환값이 있고, BP에서 사용할 경우**

- DECLARE_DYNAMIC_DELEGATE_RetVal(RetValType, FDelegateName)
- DECLARE_DYNAMIC_DELEGATE_RetVal_OneParam(RetValType, FDelegateName, Param1Type)
- ....
- DECLARE_DYNAMIC_DELEGATE_RetVal_NineParams(RetValType, FDelegateName, Param1Type, ..., Param9Type)

---

#### Multi - Cast 델리게이트 선언

**1) 반환값이 없는 경우**

- DECLARE_MULTICAST_DELEGATE(FDelegateName)
- DECLARE_MULTICAST_DELEGATE_OneParam(FDelegateName, Param1Type)
- ....
- DECLARE_MULTICAST_DELEGATE_NineParams(FDelegateName, Param1Type, ..., Param9Type)

**2) 반환값이 없고, BP에서 사용할 경우**

- DECLARE_DYNAMIC_MULTICAST_DELEGATE(FDelegateName)
- DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FDelegateName, Param1Type)
- ....
- DECLARE_DYNAMIC_MULTICAST_DELEGATE_NineParams(FDelegateName, Param1Type, ..., Param9Type)

**3) 반환값이 있는 경우**

반환값이 있는 델리게이트는 Single - Cast만 선언할 수 있다.

---

#### 델리게이트 유형에 따른 함수 선언

**1) 반환값이 없는 경우**

- void Function()
- void Function(<Param1>)
- ...
- void Function(<Param1>, ..., <Param9>)

**2) 반환값이 있는 경우**

- <RetVal> Function()
- <RetVal> Function(<Param1>)
- ...
- <RetVal> Function(<Param1>, ..., <Param9>)

---

#### 함수 바인딩

구현된 특정 함수와 델리게이트를 바인딩 해주는 함수이다

**1) SingleCast**

- Bind() : 기존 델리게이트 오브젝트 바인딩
- BindUObject() : UObject 기반 멤버 함수 델리게이트 바인딩 
- UnBind() : 델리게이트 바인딩 해제

**2) MultiCast**

- Add() : 기존 델리게이트 오브젝트 바인딩
- AddUObject() : UObject 기반 멤버 함수 델리게이트 바인딩 
- Remove() : 델리게이트 실행 목록에서 함수 제거

---

#### 델리게이트 호출

바인딩 된 함수를 호출하는 함수이다.

**1) SingleCast**

- Execute()
- ExecuteIfBound()
- IsBound()

**2) MultiCast**

- Broadcast()