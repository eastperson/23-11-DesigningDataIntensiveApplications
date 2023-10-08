
# 분산 데이터

- 확장성: 부하 분산
- 내결함성 / 고가용성: 장비 하나에 문제가 발생해도 다른 장비 사용
- 지연시간: 지리적으로 가까운 곳에 데이터 센터 위치

# 고부하로 확장
- 공유 메모리 아키텍처: 스케일 업
- 비공유 아키텍처: 스케일 아웃
	- 노드 간 통신이 필요해서 복잡해짐

복제 대 파티셔닝
- 복제: 같은 데이터의 복사본을 다른 여러 노드에 유지함, 중복성을 제공함
- 파티셔닝: 큰 데이터베이스를 파티션이라는 작은 서브넷으로 나누고 각 파티션을 각기 다른 노드에 할당(샤딩)


# 5장: 복제

복제가 필요한 이유
- 확장성: 부하 분산
- 내결함성 / 고가용성: 장비 하나에 문제가 발생해도 다른 장비 사용
- 지연시간: 지리적으로 가까운 곳에 데이터 센터 위치시켜 지연 시간을 줄임

노드 간 변경을 복제하기 위한 3가지 인기 있는 알고리즘
1. 단일 리더
2. 다중 리더
3. 리더 없는 복제

복제에서 고려해야할 트레이드오프
- 동기식 vs 비동기식
- 잘못된 복제본의 처리

*최종적 일관성: 시간이 지나면 결국은 노드들의 데이터가 일치하게됨

## 5.1. 리더와 팔로워

리더 기반으로 복제하는 방법, Active / Passive, Master / Slave 라고도 함
- 노드 중 하나를 리더로 지정하고 리더가 데이터를 쓰기 할 때마다 복제 로그나 변경 스트림을 팔로워에게 전송함
- 각 팔로워는 로그를 받아 리더가 처리한 것과 동일한 순서로 모든 쓰기를 적용함

### 5.1.1. 동기식 대 비동기식 복제

- 복제가 동기식으로 비동기식으로 일어나는지를 지정 (보통은 관계형 데이터베이스의 옵션)
- 동기식은 팔로워와 리더가 동일한 데이터를 가지고 있는 것을 보장할 수 있음
- 하지만 팔로워가 죽거나 네트워크 문제로 응답하지 않으면 쓰기 작업이 처리 될 수 없음

- 보통 리더 기반 복제는 비동기식으로 구성

### 5.1.2. 새로운 팔로워 설정

- 복제 수를 늘리거나 장애 노드 대체 시 새로운 팔로워 설정이 필요

새로운 팔로워가 리더의 데이터 복제본을 정확히 가지고 있는지 보장하는 방법
1. 스냅샷 데이터를 팔로워 노드에 복사
2. 팔로워는 리더에 연결해 스냅샷 이후에 발생한 모든 데이터 변경을 요청
- PostgreSQL: log sequence
- MySQL: Binlog
3. 팔로워가 스냅샷 이후 데이터 변경의 미처리분을 모두 처리하면 OK
	
### 5.1.3. 노드 중단 처리

- 개별 노드의 장애에도 전체 시스템이 동작하게끔 노드 중단의 영향을 최소화

#### 5.1.3.1. 팔로워 장애: 따라잡기 복구

- 팔로워는 연결이 끊어진 이후에 발생한 데이터 변경을 리더로부터 싱크 받음 (심플))

#### 5.1.3.2. 리더 장애: 장애 복구

까다로움
- 리더가 장애인지 판단하는 기준 (ex. 지연시간 30초?)
- 팔로워 중 하나를 리더로 승격 해야함
- 클라이언트는 쓰기 요청 시 새로운 리더에 요청해야함


팔로워가 이전 리더의 모든 데이터를 수신했다고 보장할 수 있는가?

### 5.2.3. 복제 로그 구현

#### 5.2.3.1. 구문 기반 복제
```
Insert ....
```

#### 5.2.3.2. 쓰기 전 로그 배송

#### 5.2.3.3. 논리적(로우 기반) 로그 복제

3가지 케이스에 따라 필요 정보가 다름
- 추가: 삽입된 로그는 모든 컬럼의 새로운 값을 포함
- 삭제: 로우를 고유하게 식별하는데 필요한 정보를 포함
- 업데이트: 로우를 고유하게 식별하는데 필요한 정보와, 업데이트 된 모든 컬럼의 새로운 값


예: MySQL Binlog
- CDC (Change Data Capture)에 활용 가능
- 논리적 로그 형식은 외부 애플리케이션에서 파싱하기 쉬움 
```
(a, 1, 2, 3, 4)
```

#### 5.2.3.4. 트리거 기반 복제

- 데이터베이스 시스템에서 데이터가 변경되면 자동으로 실행
- 오버헤드가 큼

## 5.2. 복제 지연 문제

- 아직 동기화 되지않은 팔로워의 데이터를 읽을 수 있음
- 최종적 일관성: 모든 노드의 데이터는 시간이 지나면 결국 동일해짐
	- 여기서 시간이 얼마나 걸리는지가 모호함


### 5.2.1. 자신이 쓴 내용 읽기

- 사용자가 수정한 내용을 읽을 때는 리더에 읽음
- 타임스탬프 활용

페이스북 사례
- 자신의 정보는 변경이 가능하기에 쓰기 노드에서 읽기
- 다른 사람의 정보는 변경이 불가능하니 읽기 노드에서 읽기


### 5.2.2. 단조 읽기


단조 읽기
- 시간 역전 현상을 극복하기 위한 방법
- 강한 일관성보다는 덜한 보장이지만 최종적 일관성보다는 더 강한 보장임


각 사용자의 읽기가 항상 동일한 복제 서버에서 수행되도록 해서 시간 역전 현상 극복
- 쿼리할 노드를 선택할 때 임의 선택보다는 사용자 ID의 해시를 기반으로 선택
- 쿼리 대상 노드가 고장나면 쿼리를 다른 노드로 라우팅이 필요함

### 5.2.3. 일관된 순서로 읽기

- 인과성이 있는 쓰기는 동일한 파티션에 기록되도록 함

### 5.2.4. 복제 지연을 위한 해결책

- 복제가 비동기식으로 동작하지만 동기식으로 동작하는 것이 문제 해결 방안
- 애플리케이션 코드에서 복제 지연 문제를 다루기에는 복잡도가 높아 잘못되기 쉬움
- 데이터베이스를 신뢰할 수 있다면 개발자는 복잡한 복제 문제를 걱정하지 않고 개발할 수 있음
	- 트랜잭션이 있는이유, 트랜잭션은 애플리케이션이 더 단순해지기 위해 데이터베이스가 더 강력한 보장을 제공하는 방법


## 5.3. 다중 리더 복제

단일 리더 기반 복제는 리더에 장애가 생길 경우 쓰기를 할 수 없음

### 5.3.1. 다중 리더 복제의 사용 사례

- 단일 데이터센터 내에서의 다중 리더 설정은 추가된 복잡도에 비해 이점이 크지 않음

#### 5.3.1.1. 다중 데이터 센터 운영

- 성능: 지리적으로 가까운 로컬 데이터센터의 존재로 인해 데이센터 간 네트워크 지연은 사용자에게 숨겨짐
- 데이터센터 중단 내성
- 네트워크 문제 내성


#### 5.3.1.2. 오프라인 작업을 하는 클라이언트
- 캘린더 앱
- 카우치DB

#### 5.3.1.3. 협업 편집

- 구글 Docs

### 5.3.2. 쓰기 충돌 다루기

#### 5.3.2.1. 동기 대 비동기 충돌 감지

#### 5.3.2.2. 충돌 회피

- 특정 레코드의 모든 쓰기는 동일한 리더를 거치도록 함

#### 5.3.2.3. 일관된 상태 수렴

- 쓰기를 받은 순서대로 적용하게되면 리더 마다 데이터가 다를 수 있음

일관된 상태 수렴 방법
- 각 쓰기에 고유 ID 부여
- 어떻게든 병합
- 명시적 데이터 구조에 충돌을 기록해 모든 정보를 보존

### 5.2.3. 다중 리더 복제 토폴로지


--- 

데이터가 일치하지 않는 경우가 있었는지
- 없었음 ㅠㅠ

통제할 수 있는 Saga
통제할 수 없는 Third-Party