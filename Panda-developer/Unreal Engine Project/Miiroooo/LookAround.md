```cpp
	FVector2D LookVector = Value.Get<FVector2D>();
	AddControllerYawInput(LookVector.X);
	AddControllerPitchInput(LookVector.Y); 
	
	float Pitch = LookVector.Y + CurrentPitch;
	CurrentPitch = FMath::Clamp(Pitch, -45.0f, 45.0f);	
```

 <span style="color:rgb(188, 149, 218)">||</span> _`FVector2D LookVector = Value.Get<FVector2D>();`_
 
 LookVector은 입력된 마우스 이동 값을 가져오는 벡터이다.
 2D는 (x, y) 값을 가지며 x는 좌우, y는 상하 값을 나타낸다. (4사분면)
<br>

<span style="color:rgb(188, 149, 218)">||</span> _`AddControllerYawInput(Value);`_ 

마우스 좌우 이동 값에 따라 플레이어(컨트롤러) 좌우 회전 조절
#Yaw : z축 기준 회전 (좌우)
<br>

<span style="color:rgb(188, 149, 218)">||</span> _`AddControllerPitchInput(Value);`_

마우스 상하 이동 값에 따라 플레이어(컨트롤러) 상하 회전 조절
#Pitch : y축 기준 회전 (상하)

<span style="color:rgb(255, 0, 0)">2D 선상에서의 Roll, Pitch, Yaw
3D 선상에서의 Roll, Pitch, Yaw</span> 
<br>

<span style="color:rgb(188, 149, 218)">||</span> _`float Pitch = LookVector.Y + CurrentPitch;`_
<span style="color:rgb(188, 149, 218)">||</span> _`CurrentPitch = FMath::Clamp(Pitch, -45.f, 45.f);`_

마우스 상하 값이 누적되도록 적용

Pitch값이 -45도 ~ 45도 사이를 넘지 않도록 #Clamp 함수를 통해 범위를 제한해준다.
>**<span style="color:rgb(255, 192, 0)">//</span> Current Pitch를 따로 저장하는 이유**
>
> 후에 수류탄 던지는 기능 등 상하 회전 각도값을 사용 할 일을 대비하여 변수 생성. 
>
>