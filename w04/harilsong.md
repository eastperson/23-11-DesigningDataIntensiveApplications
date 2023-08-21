# Ch3 저장소와 검색 (데이터 베이스를 강력하게 만드는 데이터 구조)

- 로그 구조(log-structured) 저장소 엔진
- 페이지 지향(page-oriented) 저장소 엔진

## 데이터베이스를 강력하게 만드는 데이터 구조

```shell
#!/bin/bash

db_set() {
  echo "$1,$2" >> database
}

db_get() {
  grep "^$1," database | sed -e "s/^$1,//" | tail -n 1
}
```

```shell
$ db_set 123456 '{"name": "Haril Song", "age": 25}'
$ db_set 234567 '{"name": "Gildong Hong", "age": 31}'

$ db_get 123456
{"name": "Haril Song", "age": 25}
```

기본적인 저장소 형식은 매우 간단하다. db_set 을 호출할 때마다 파일의 끝에 추가하므로 키를 여러 번 갱신해도 값의 예전 버전을 덮어쓰지 않는다. 최신 값을 찾기 위해서는 파일에서 키의 마지막 항목을 살펴봐야
한다(그래서 db_get 에서 tail -n 1 을 사용한다).

일반적으로 파일 추가 작업은 매우 효율적

많은 데이터베이스는 내부적으로 추가 전용 데이터 파일인 로그(log)를 사용한다.

반면 db_get 함수는 데이터베이스에 많은 레코드가 있으면 성능이 매우 좋지 않다. 데이터베이스가 커지면 커질수록 파일을 처음부터 끝까지 읽어야 하기 때문이다. 이런 문제를 해결하기 위해 데이터베이스는 인덱스를
사용한다.

색인은 기본 데이터에서 파생된 **추가적인** 구조다. 단순히 파일에 추가하는 작업이 가장 간단한 쓰기 작업이기 때문에, 어떤 종류의 색인을 사용하더라도 가장 간단한 쓰기 작업 성능을 앞서기는 어렵다.

색인을 전략적으로 사용해야 읽기 성능을 향상시키면서 불필요한 쓰기 오버헤드를 감소시킬 수 있다.

### 해시 색인

인메모리 데이터 구조 활용

파일에 항상 추가만 한다면 결국 디스크 공간이 부족해진다. 특정 크기의 세그먼트(segment)로 로그를 나누는 방식을 사용할 수 있다. 특정 크기에 도달하면 segment 파일을 닫고 새로운 세그먼트 파일에 이후
쓰기를 수행한다. 한 번 세그먼트가 분리되면 수정할 수 없기 때문에 병합 등의 압축과정이 유리해진다. 병합 과정이 끝난 이후의 읽기 요청은 이전 세그먼트 대신 새로 병합한 세그먼트를 사용하게끔 전환한다. 전환 후에는
이전 세그먼트 파일을 삭제하여 디스크 공간을 다시 확보할 수 있다.

### SS 테이블과 LSM 트리

> 정렬된 문자열 테이블 (Sorted String Table, SSTable)

이전 세그먼트 파일에서 일련의 키-값 쌍을 키로 정렬하는 간단한 요구사항이 추가된 것이다. 언뜻 보기에 이 요구사항은 순차 쓰기를 사용할 수 없게 만드는 것 같다. 하지만 SS 테이블은 해시 색인을 가진 로그
세그먼트보다 몇 가지 큰 장점이 있다.

1. 세그먼트 병합은 파일이 사용 가능한 메모리보다 크더라도 간단하고 효율적이다.
2. 파일에서 특정 키를 찾기 위해 더는 메모리에 모든 키의 색인을 유지할 필요가 없다. 키가 정렬되어 있기 때문에 이분탐색으로 금방 찾을 수 있다.
3. 디스크 공간을 절약하고 I/O 대역폭 사용도 줄인다.

#### SS 테이블 생성과 유지

유입되는 쓰기는 임의 순서로 발생하는데 어떻게 데이터를 키로 정렬할 수 있을까?

레드 블랙 트리나 AVL 트리와 같은 데이터 구조를 이용하면 임의 순서로 키를 삽입하고 정렬된 순서로 해당 키를 다시 읽을 수 있다.

이제 저장소 엔진을 다음과 같이 만들 수 있다.

- 쓰기가 들어오면 인메모리 균형 트리(balanced tree) 데이터 구조에 추가한다. 이 인메모리 트리는 멤테이블(memtable)이라고도 한다.
- 멤테이블이 특정 크기에 도달하면 디스크에 SS 테이블 파일로 기록한다.
- 읽기 요청을 제공하려면 먼저 멤테이블에서 키를 찾아야 한다. 그 다음 디스크 상의 가장 최신 세그먼트에서 찾는다.
- 가끔 세그먼트 파일을 합치고 덮어 쓰여지거나 삭제된 값을 버리는 병합과 컴팩션 과정을 수행한다.

데이터베이스가 고장났을 때 아직 디스크에 기록되지 않은 멤테이블은 유실될 수 있다. 이런 문제를 피하기 위해서 이전 절과 같이 매번 쓰기를 즉시 추가할 수 있게 분리된 로그를 디스크 상에 유지해야 한다. 이 로그는
손상 후 멤테이블을 복원할 때만 필요하기 때문에 순서가 정렬되지 않아도 문제되지 않는다. 멤테이블을 SS 테이블로 기록하고 나면 해당 로그는 버릴 수 있다.

#### LSM 트리

#### 성능 최적화

- 존재하지 않는 키값을 찾기 위해 세그먼트를 풀스캔하는 경우를 방지하기 위해 블룸 필터를 사용할 수 있다.
- 크기 계층과 레벨 컴팩션

### B 트리

B 트리는 SS 테이블과 같이 키로 정렬된 키-값 쌍을 유지하기 때문에 키-값 검색과 범위 질의에 효율적이다. 하지만 B 트리는 LSM 트리와는 설계 철학이 매우 다르다.

B 트리는 전통적으로 4KB 크기의 고정 크기 블록이나 페이지로 나누고 한 번에 하나의 페이지에 읽기 또는 쓰기를 한다. 디스크가 고정 크기 블록으로 배열되기 때문에 이런 설계는 근본적으로 하드웨어와 조금 더 밀접한
관련이 있다.

B 트리의 한 페이지에서 하위 페이지를 참조하는 수를 **분기 계수(branching factor)** 라고 부른다.

#### 신뢰할 수 있는 B 트리 만들기

#### B 트리 최적화

- 쓰기 시 복사 방식 사용
- 페이지에 전체 키를 저장하는게 아니라 키를 축약해 쓰면 공간을 절약할 수 있다.
- 리프 페이지를 디스크 상에 연속된 순서로 나타나개끔 트리를 배치하려 시도한다.
- 트리에 형제 페이지에 대한 참조 포인터를 추가한다.

### B 트리와 LSM 트리 비교

| 기능 | B 트리 | LSM 트리 |
|----|------|--------|
| 읽기 | 빠름   | 느림     |
| 쓰기 | 느림   | 빠름     |

LSM 트리에서 읽기가 느린 이유는 각 컴팩션 단계에 있는 여러가지 데이터 구조와 SS 테이블을 확인해야 하기 때문이다.

### 기타 색인 구조

- 보조 색인(secondard index): 효율적으로 조인을 수행하는 데 결정적인 역할을 한다.

#### 색인 안에 값 저장하기

- 클러스터드 색인: 색인 안에 바로 색인된 로우를 저장한다. MySQL 의 PK 는 언제나 클러스터드 색인이고 보조 색인은 기본키를 참조한다.
- 비클러스터드 색인: 색인 안에 로우를 저장하지 않고 로우의 위치를 가리키는 포인터만 저장한다.
- 커버링 색인: 절충안, 색인 안에 테이블의 칼럼 일부를 저장한다. 이렇게 하면 색인만 사용해 일부 질의에 응답이 가능하다.

#### 다중 칼럼 색인

지금까지 설명한 색인은 하나의 키만 값에 대응한다. 이 방식은 테이블의 당중 칼럼에 동시에 질의를 해야 한다면 충분하지 않다.

결합 색인은 하나의 칼럼에 다른 칼럼을 추가하는 방식으로 하나의 키에 여러 필드를 단순히 결합한다. 이 방법은 색인 정의시의 칼럼의 순서에 동작이 의존하게 된다.

다차원 색인은 한 번에 여러 칼럼에 질의하는 조금 더 일반적인 방법이다. 특히 지리 공간 데이터에 중요하게 사용된다. PostGIS 에서는 R 트리처럼 전문 공간 색인을 사용한다. 일반적인 색인에서는 위도, 경도 둘 중 하나의 정보로만 질의할 수 있지만 이런 색인을 사용하면 두 정보를 동시에 사용해 질의할 수 있다.

#### 전문 검색과 퍼지 색인

- 트라이
- 레벤슈타인 오토마톤

#### 모든 것을 메모리에 보관

- 인메모리 데이터베이스
- 비휘발성 메모리