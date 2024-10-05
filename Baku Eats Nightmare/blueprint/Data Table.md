### #DataTable 
Weapon에 대한 DATA TABLE을 만드는 과정을 예시로 나타내었다.

<br>

#### 1. #Enumeration
무기들의 종류(유형)을 정의해준다. 종류에 따라 다른 동작(발사 속도, 반동, 애니메이션 효과 등)을 구현할 수 있다.
ex) 권총 종류의 총 3개, 장거리용 종류의 총 2개 이런식으로 관리
![[Pasted image 20241005183214.png|300]]

#### 2. #Structure
Data Table을 만들 때 속성들에 대한 타입을 지정해준다. 
아래 사진은 DataTable의 속성으로 (무기이름, 무기 종류, 데미지, 탄약 최대 수, 땅에 떨어진 무기 BP, 주워서 손에 장착하는 무기 BP) 를 지정해준 것이다. 
![[Pasted image 20241005183323.png]]

#### 3. #DataTable
빈곳 우클릭을 통해 Data Table 을 만들어 줄것이다. 이때 이전에 만들어둔 Structure를 바탕으로 생성할 수 있다. 그래서 위에 만들어 두었던 Structure를 가져와 Row를 자동으로 생성해준다.
![[Pasted image 20241005183554.png|300]]
![[Pasted image 20241005183632.png|300]]
![[Pasted image 20241005183737.png]]

#### 4. BP_PickUpWeaponMaster 에 Data Table 추가
모든 무기들은 Data Table을 가지고 있으므로 WeaponData를 추가해준다. 이때 변수의 타입은 #DataTableRowHandle 이다.
![[Pasted image 20241005183917.png]]

변수를 추가해주면 Details에 Data Table 정보를 수정할 수 있는 UI가 생긴것을 확인할 수 있다.
![[Pasted image 20241005184029.png]]
