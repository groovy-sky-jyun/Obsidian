캐릭터의 다양한 움직임을 사용하기 위해 무료 에셋을 제공해주는 #Mixamo 홈페이지를 사용할 것이다. Mixamo의 애니메이션을 언리얼에 적용시키기 위해서는 #MixamoConverter 를 사용해 주어야 한다. 블렌더를 사용해줘도 되지만 Mixamo Converter를 사용해 주는게 훨씬 간편하기 때문에 이 방법을 선택했다. 

#### *Mixamo Converter를 사용하는 이유*

>*과연 Mixamo에서 그냥 다운로드한 에셋을 바로 언리얼에 import하게 되면 어떻게 될까?*
>
>- <span style="color:rgb(255, 92, 92)">mesh contains root bone as root but animation doesn't contain the root track. import failed </span>라는 오류가 발생하게 된다.
>
이는 import한 애니메이션의 구조와 스켈레톤의 구조가 일치하지 않아서 발생하는 오류이다. 스켈레톤에는 root bone이 있지만 애니메이션에는 루트가 없는 상태로 import가 되는것이다. 
>
그래서 우리는 <span style="color:rgb(193, 173, 240)">mixamo converter를 통해 mixamo에 bone을 붙여주고 그 위에 원하는 애니메이션 에셋을 덧붙여주는 방식</span>을 사용한 것이다.

<br>

## Mixamo Converter 설치
Mixamo Converter를 설치하기 위해서는 [Mixamo Converter](https://terribilisstudio.fr/?section=LAUNCHER) 홈페이지에 방문해서 Terribilis studio launcher를 설치해준 뒤 런처에서 coverter를 설치해주어야 한다.  
![[Pasted image 20240802215531.png|500]]

설치가 완료되고 런처를 실행시킨 뒤 Mixamo Converter를 설치한 다음 실행시켜준다.
![[Pasted image 20240802215808.png|500]]
![[Pasted image 20240802215920.png|500]]

실행화면에서 Enter the conversion process를 클릭해준다. 
![[Pasted image 20240802215953.png|500]]

그러면 캐릭터를 선택할 수 있는데 이 캐릭터가 바로 bone 캐릭터가 될 것이다. 나는 UE5를 사용하고 있으므로 Manny를 선택해준다.
![[Pasted image 20240802220038.png|500]]

그러면 해당 bone 캐릭터 파일이 있는 폴더가 열린다. 이 파일을 사용해주어야 하므로 위치를 기억하거나 폴더를 닫지 않고 다음 단계를 실행할 것이다.
![[Pasted image 20240802220144.png|500]]

<br>

## Mixamo Animation Download
이제 Mixamo 홈페이지에서 원하는 애니메이션을 다운받아 줄 것이다. 우선은 [Mixamo Animation](https://www.mixamo.com/#/?page=1&type=Motion%2CMotionPack)에 들어가서 Mixamo Converter에서 다운받은 bone 캐릭터를 적용시켜줄 것이다. 
 1. UPLOAD CHARACTER 버튼을 눌러준다.
![[Pasted image 20240802220630.png|500]]
<br>
2. Mixamo Converter에서 연 폴더에서 _SKM_Manny.FBX 파일을 업로드 위치에 끌어당겨 놓는다.
![[Pasted image 20240802220756.png|500]]
<br>
3. 캐릭터가 잘 업로드 된 것을 확인하고 NEXT 버튼을 눌러준다.
![[Pasted image 20240802221107.png|500]]

4. 원하는 애니메이션을 선택한 뒤 DOWNLOAD 버튼을 눌러준다.
![[Pasted image 20240802221214.png|500]]

> ###### 선택한 애니메이션이 <mark style="background: rgb(193, 173, 240)">단일 애니메이션</mark>일 경우
> 아래와 같이 설정을 해준 뒤 다운로드 해준다.
> ![[Pasted image 20240802222444.png|500]]
> ###### 선택한 애니메이션이 <mark style="background: rgb(193, 173, 240)">Package 애니메이션</mark>일 경우
> 아래와 같이 설정을 해준 뒤 다운로드 해준다.
> ![[Pasted image 20240802222234.png|500]]

<br>

## Mixamo Converter 사용하여 변환
다운로드 받은 애니메이션이 Package일 경우 폴더 압축을 해제해주어야 한다. 압축해제할 때 폴더 위치를 <span style="color:rgb(193, 173, 240)">TerribilisLauncher > mixamo_converter > IncomingFbx</span> 로 지정해서 압축을 풀어준다. Package가 아니라면 압축해제없이 파일을 바로 폴더에 이동시키면 된다.
![[Pasted image 20240802223018.png|500]]

>###### Mixamo_converter 폴더
>- IncomingFbx : 변환하고자 하는 파일을 넣음
>- OutgoingFbx : 변환된 파일이 생성되는 폴더
>![[Pasted image 20240802223259.png|500]]

<br>

알맞은 위치에 압축을 해제해준 뒤 mixamo converter에 아까 전 bone 캐릭터를 선택하는 화면에서 <span style="color:rgb(193, 173, 240)">스크롤을 아래</span>로 내려보면 넣은 파일명이 뜨는것을 확인할 수 있다. 
1. _(optional)_ 부분에는 .UE5로 입력해주었다. (자신이 사용하는 버전의 숫자를 입력하면 된다.)
2. 그런 다음 <span style="color:rgb(193, 173, 240)">Click here to convert the animations</span> 버튼을 누른다.
![[Pasted image 20240802224248.png|500]]
3. 그러면 변환이 진행되고 완료되면 오른쪽에 변환된 파일명이 뜬다. <span style="color:rgb(193, 173, 240)">Open the folder with converted animation</span> 버튼을 누르면 OutgoingFbx 폴더가 열린다.
![[Pasted image 20240802224115.png|500]]

<br>

## 언리얼에 Animation Import
애니메이션을 import 하고자 하는 위치에 들어간 뒤 Import 버튼을 눌러 원하는 애니메이션을 선택해준다. __이때 Import는 <span style="color:rgb(193, 173, 240)">TerribilisLauncher > mixamo_converter > OutgoingFbx</span>에 있는 파일을 사용해야 한다.__
![[Pasted image 20240802224903.png|600]]

그러면 <span style="color:rgb(193, 173, 240)">FBX Import Options</span> 창이 뜨게 되며 아래의 사진과 같이 설정해준다. <span style="color:rgb(193, 173, 240)">Skeleton</span>은 이전에 bone 캐릭터를 Manny로 해줬기 때문에 <span style="color:rgb(193, 173, 240)">SK_Mannequin</span>을 선택해주었다.
![[Pasted image 20240802225132.png|400]]

이제 Import를 하게 되면 잘 등록된것을 확인할 수 있다.
![[Pasted image 20240802225504.png|600]]
