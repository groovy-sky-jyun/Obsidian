<br>

## ABP_Player 

IA_LookAround(InputAction)에 대한 동작을 설정해 줄 것이다.

1. 마우스의 y 축 값에 따라 캐릭터의 등이 굽어졌다 젖혀졌다한다.

<br>

#### 1. Player Character 변수 설정
#GetOwningActor 를 통해서 현재 캐릭터의 액터를 가져온 뒤 이를 #CastTo 를 통해서 특정 타입 클래스로 변환시켜준다. 

캐스팅을 하는 이유는 캐스팅을 해야만 그 클래스에 존재하는 변수나 함수에 접근할 수 있기 때문이다.

캐스팅 노드를 통해 리턴값을 변수로 만들어준다. 
![[Pasted image 20240930204613.png|300]]
![[Pasted image 20240930205709.png|300]]
![[Pasted image 20240930204402.png|ABP Character의 Event Graph]]

<br>

#### 2. Blueprint Thread Safe Update Animation
Blueprint Thread Safe Update Animation를 통해 애니메이션의 업데이트를 병렬로 처리하도록 한다. 
![[Pasted image 20240930204724.png]]

#PropertyAccess 노드를 통해서 BP_Character의 변수에 접근한다. 거기에서 만들어 둔 Pitch 변수를 가져와 ABP 클래스의 변수로 Set한다.
![[Pasted image 20240930205806.png|300]]
![[Pasted image 20240930205903.png|400]]
![[Pasted image 20240930210041.png|300]]

<br>

#### 3. AnimGraph를 통해 뼈 구부리기
우선은 Idle, Walk, Jump, Land 등을 관리하는 Locomotion StateMachine을 캐싱 해준다. 이는 해당 StateMachine을 재사용 할 수 있게 하기 위함이다.
![[Pasted image 20240930211155.png]]

그런 뒤 #UseCachedPose를 통해서 Locomotion StateMachine을 가져온다. 그럼 Locomotion StateMachine 에서 값의 변화가 있으면 가장 최신 포즈를 리턴하게 된다.

해당 포즈를 #LayerdBlendPerBone 노드에 연결시킨다. 그러면 캐릭터의 일부분만 다른 애니메이션으로 변경하고 나머지 부분은 리턴받은 가장 최신 포즈를 유지하게 된다.

![[Pasted image 20240930211451.png]]


 #### <mark style="background: #D2B3FFA6;">TIPS </mark>
> #### #LayerdBlendPerBone
> 애니메이션의 포드를 뼈대 계층 구조에 따라 블렌딩 할 수 있도록 해준다.
> 
> 보통 캐릭터의 특정 부위만 다른 애니메이션을 적용하고 싶을 때 사용된다.
> 
> 상체 하체에 다른 포즈를 주거나 무기 휘두드리 처럼 일부분에만 다른 애니메이션이 적용되고 나머지는 기본 동작을 유지할 수 있도록할 수 있다.
>
>__Base pose 와 Blend Poses 0__
>Base pose는 기본이 되는 포즈이다. 위에서는 Locomotion에서 리턴되는 가장 최신 포즈가 된다.
>Blend Poses 는 기본 포즈에 덮어 씌워질 포즈이다. Blend Poses로 연결한 sequence pose에서 뼈대가 

<br>

#### 2.2 ABP Character를 통해서 등 구부리는 구현
아래 링크로 이동
[애니메이션>Animation Blueprint>캐릭터 등 구부리기](캐릭터%20등%20구부리기(마우스%20상하)_LookAroundInput.md)

<br>

#### LookAround 전체 노드
![[Pasted image 20240930203053.png]]