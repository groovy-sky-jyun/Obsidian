_크래프톤 직무테스트 출제_

### 데드락 발생 조건

(4가지 모두 성립해야한다.)

##### 1. Mutual Exclusion (상호 배제)

자원은 한번에 한 프로세스만 사용할 수 있다.

##### 2. Hold and Wait (점유 대기)

각 프로세스들이 최소한 하나의 자원을 점유하고 있다.

##### 3. No Preemption (비선점)

다른 프로세스에 할당된 자원을 강제로 빼앗을 수 없다.

##### 4. Circular Wait (순환 대기)

순환 형태로 자원을 대기하고 있어야 한다.
