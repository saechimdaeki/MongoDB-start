
## What is MongoDB ? 

기존 Database의 단점 보완을 위해 Custom Data Store 개발 -> 확장성과 신속성에서 자주 문제 발생 -> Database개발 -> `MongoDB`

```markdown
- Schema가 자유롭다
- HA와 Scale-Out Solution을 자체적으로 지원해서 확장이 쉽다
- Secondary Index를 지원하는 NoSQL이다
- 다양한 종류의 Index를 제공한다
- 응답 속도가 빠르다
- 배우기 쉽고 간편하게 개발이 가능하다
```

### `MongoDB는 유연하고 확장성 높은 Opensource Document 지향 Database이다`

---

# SQL vs NoSQL

## SQL

기존 2000년대 다음과 같은 테이블이 있다고 가정하자. 유지되던 중 sns가 도입되기 시작했다.

<img width="980" alt="image" src="https://user-images.githubusercontent.com/40031858/207834423-2cfdc3d9-58de-4c44-8080-87f0acccd843.png">

테이블은 따라서 다음과 같이 되었다.

<img width="1238" alt="image" src="https://user-images.githubusercontent.com/40031858/207834810-ecd2303b-39be-48d5-8784-655017d9afa8.png">

그래서 관계형 데이터베이스에서는 정규활르 통해 테이블을 나누기 시작.

<img width="891" alt="image" src="https://user-images.githubusercontent.com/40031858/207835017-f1f8840f-4032-4ead-a591-ec06c50c1b78.png">

### SQL 장단점
- `데이터 중복을 방지할 수 있다`
- `Join의 성능이 좋다`
- `복잡하고 다양한 쿼리가 가능하다`
- `잘못된 입력을 방지할 수 있다`
- 하나의 레코드를 확인하기 위해 여러 테이블을 Join하여 가시성이 떨어진다
- 스키마가 엄격해서 변경에 대한 공수가 크다

### Scaling에 대한 장단점
- Scale-Out이 가능하지만, 설정이 어렵다
- 확장할 때마다 App단위 수정이 필요하다
- 전통적으로 Scale-Up 위주로 확장했다.



## NoSQL

![image](https://user-images.githubusercontent.com/40031858/207836647-fa68c479-7ae2-4278-9f10-54e57dc7c7cf.png)


관계형 데이터베이스에서 다음과 같던 table

<img width="891" alt="image" src="https://user-images.githubusercontent.com/40031858/207835017-f1f8840f-4032-4ead-a591-ec06c50c1b78.png">

이 테이블을 document로 다음과 같이 표현할 수 있다

```json
{
    _id: 1,
    first_name "saechim",
    last_name: "daeki",
    zipcode: 123456,
    phone: "123-456-1234",
    sns: [
        {type: "email", id: "saechim@naver.com"},
        {type: "instagram", id: "saechimdaeki"}
    ]
}
```

### NoSQL 장단점

- `데이터 접근성과 가시성이 좋다`
- `Join 없이 조회가 가능해서 응답 속도가 일반적으로 빠르다`
- `스키마 변경에 공수가 적다`
- `스키마가 유연해서 데이터 모델을 App의 요구사항에 맞게 데이털르 수용할 수 있다`
- 데이터의 중복이 발생한다
- 스키마가 자유롭지만, 스키마 설계를 잘해야 성능 저하를 피할 수 있다.

### Scaling에 대한 장단점
- HA와 Sharding에 대한 솔루션을 자체적으로 지원하고 있어 Scale-Out이 간편하다
- 확장 시, Application의 변경사항이 없다.

### 요약
- MongoDB는 Document 지향 Database이다.
- 데이터 중복이 발생할 수 있지만, 접근성과 가시성이 좋다
- 스키마 설꼐가 어렵지만, 스키마가 유연해서 Application의 요구사항에 맞게 데이터를 수용할 수 있다
- 분산에 대한 솔루션을 자체적으로 지원해서 Scale-Out이 쉽다
- 확장 시, Application을 변경하지 않아도 된다.


---

## MongoDB 구조

||||
|:--:|:--:|:--:|
|`RDBMS`||`MongoDB`|
|Cluster| -> | Cluster|
|Database | -> | Database|
|Table| -> |Collection|
|Row| -> |Document|
|Column| -> |Field|

### 기본 Database

|||
|:--:|:--|
|`Database`| `Description`|
|admin| - 인증과 권한 부여 역할 <br/> - 일부관리 작업을 하려면 admin Database에 대한 접근이 필요하다|
|local|- 모든 mongod instance는 local database를 소유한다 <br/> - oplog와 같은 replication 절차에 필요한 정보를 저장한다 <br/> - startup_log와 같은 instance 진단 정보를 저장한다 <br/> - local database 자체는 복제되지 않는다|
|config|- shared cluster에서 각 shard의 정보를 저장한다|


![image](https://user-images.githubusercontent.com/40031858/208274651-1bad092c-a052-4289-a136-32e7b744ffd7.png)

### Collection 특징
- 동적 스키마를 갖고 있어서 스키마를 수정하려면 필드 값을 추가/수정/삭제하면 된다
- Collection 단위로 Index를 생성할 수 있다
- Collection 단위로 Shard를 나눌 수 있다.

![image](https://user-images.githubusercontent.com/40031858/208275206-7dd4868d-e546-4707-af46-509da0e01cf1.png)

### Document 특징
- JSON 형태로 표현하고 BSON(Binary JSON) 형태로 저장한다
- 모든 Document에는 "_id"필드가 있고, 없이 생성하면 ObjectId 타입의 고유한 값을 저장한다
- 생성 시, 상위 구조인 Database나 Collection이 없다면 먼저 생성하고 Document를 생성한다
- Document의 최대 크기는 16MB이다

![image](https://user-images.githubusercontent.com/40031858/208275921-937dbf36-7020-4aad-8a28-d5c34fcb56b9.png)

```markdown
- Database -> Collection -> Document -> Field 순으로 구조가 형성되어 있다
- admin, config, local database는 MongoDB를 관리하는데 사용된다.
- Collection은 동적 스키마를 갖느다
- Document는 JSON형태로 표현되고 BSON 형태로 저장된다
- Document는 고유한 "_id" 필드를 항상 갖고 있다
- Document의 최대 크기는 16MB로 고정되어 있다
```

## MongoDB 배포 형태 소개

- standalone
- Replica Set
- Sharded Cluster

이러한 세가지 형태의 배포가 있다.

### standalone

<img width="788" alt="image" src="https://user-images.githubusercontent.com/40031858/208394088-3a91deb1-d547-4866-bd8b-e7894cc7e00b.png">


### Replica Set

<img width="944" alt="image" src="https://user-images.githubusercontent.com/40031858/208394175-90f7598b-0097-46ba-8a48-692065b451bb.png">

### Sharded Cluster

<img width="892" alt="image" src="https://user-images.githubusercontent.com/40031858/208394225-a4d7500d-73ed-45ba-8934-c6b6ada5b8da.png">


---

## Replica Set

### Replica Set Members

|||
|:--:|:--|
|status|Description|
|Primary|- Read/Write 요청 모두 처리할 수 있다 <br/> - Write를 처리하는 유일한 멤버이다 <br/> - Replica Set에 하나만 존재할 수 있다|
|Secondary|- Read에 대한 요청만 처리할 수 있다 <br/> - 복제를 통해 Primary와 동잃한 데이터 셋을 유지한다 <br/> - Replica Set에 여러개가 존재할 수 있다|

### Replica Set Election(Fail-Over)

<img width="595" alt="image" src="https://user-images.githubusercontent.com/40031858/208396027-1f3fb016-0c82-44a2-8f92-de923c616411.png">


### Replica Set Arbiter

<img width="1299" alt="image" src="https://user-images.githubusercontent.com/40031858/208400425-23528d92-8645-4533-b69c-2d67534833e6.png">

### Replica Set Oplog

<img width="770" alt="image" src="https://user-images.githubusercontent.com/40031858/208400533-a4f10441-c075-4ed5-9bf3-0e0b1c39867c.png">


```markdown
- Replica Set은 HA 솔루션이다.
- 데이털르 들고 있는 멤버의 상태는 Primary와 Secondary가 있다.
- Secondary는 선출을 통해 과반수의 투표를 얻어서 Primary가 될 수 있다.
- Arbiter는 데이터를 들고 있지 않고 Primary 선출 투표에만 참여하는 멤버이다.
- Replica Set은 local database의 Oplog Collection을 통해 복제를 수행한다.
```

--- 

## Sharded Cluster


### 모든 Shard는 Replica Set으로 구성되어 있다

<img width="1205" alt="image" src="https://user-images.githubusercontent.com/40031858/208403838-1c26fff7-e2ce-4152-977e-d93d05197fcf.png">

### Sharded Cluster 장단점

|||
|:--|:--|
|장점|단점|
|- 용량의 한계를 극복할 수 있다. <br/> - 데이터 규모와 부하가 크더라도 처리량이 좋다 <br/> - 고가용성을 보장한다 <br/> - 하드웨어에 대한 제약을 해결할 수 있다|- 관리가 비교적 복잡하다 <br/> - Replica Set과 비교해서 쿼리가 느리다|

### 용어 비교

|||
|:--|:--|
|Sharding|Partitioning|
|- 하나이ㅡ 큰 데이터를 여러 서브셋으로 나누어 여러 <br/> 인스턴스에 저장하는 기술|- 하나의 큰 데이터를 여러 서브셋으로 나누어 하나의 <br/> 인스턴스의 여러 테이블로 나누어 저장하는 기술 |

|||
|:--|:--|
|Replica Set|Sharded Cluster|
|- Replica Set은 각각 멤버가 같은 데이터 셋을 갖는다|- Sharded Cluster는 각각 Shard가 다른 <br/> 데이터의 서브셋을 갖는다|

<img width="815" alt="image" src="https://user-images.githubusercontent.com/40031858/208407047-510414d0-802b-4063-a15e-eb89515216b6.png">


### Sharding Collections

<img width="532" alt="image" src="https://user-images.githubusercontent.com/40031858/208407188-dc529613-f06a-470b-9d80-597d5f59a286.png">

### Ranged Sharding

<img width="877" alt="image" src="https://user-images.githubusercontent.com/40031858/208407251-c2c7cb0c-4017-4b58-aec5-7e1841eae145.png">

### Hashed Sharding

<img width="895" alt="image" src="https://user-images.githubusercontent.com/40031858/208407335-51ecda0c-8a70-41db-b891-23b364e2d67b.png">

### Zone Sharding

<img width="951" alt="image" src="https://user-images.githubusercontent.com/40031858/208407419-9c83e378-3739-42fd-98d3-6621f1be0f36.png">

```markdown
- Sharded Cluster는 MongoDB의 분산 Solution이다.
- Collection 단위로 Sharding이 가능하다.
- Sharding은 Shard Key를 선정해야하고 해당 필드에는 Index가 만들어져 있어야한다.
- 꼭 Router를 통해 접근한다.
- Range와 Hashed Sharding 두 가지 방법이 있다.
- 가능하면 Hashed Sharding을 통해 분산한다
```

---

## Replica Set vs Sharded Cluster 어떻게 배포할까?


![image](https://user-images.githubusercontent.com/40031858/208585607-98ba0f8d-164e-4682-ad75-afad4ca75c6a.png)


### 예시 1) 매일 1GB씩 데이터가 증가하고 3년간 보관 -> `환경에 따라 다르다`

### 예시 2) Write 요청이 압도적으로 많은 서비스 -> `Sharded Cluster` 
- Replica Set은 Write에 대한 분산이 불가능

### 예시 3) 논리적인 데이터베이스가 많은 경우 -> 여러 Replica Set으로 분리

### Replica Set vs Sharded Cluster

||||
|:--:|:--|:--|
|배포 형태 | 장점| 단점 |
|Replica Set|- 운영이 쉽다 <br/> - 장애 발생시 문제 해결 및 복구가 쉽다 <br/> - 서버 비용이 적게 든다 <br/> - 성능이 좋다 <br/> - 개발 시 설계가 용이하다|- Read에 대한 분산이 가능하지만, Write에 대한 분산은 불가능|
|Sharded Cluster|- Scale-Out이 가능하다 <br/> - Write에 대한 분산이 가능하다|- Replica Set의 모든 장점이 상대적으로 단점이 된다|

### 결론

![image](https://user-images.githubusercontent.com/40031858/208585998-bc63f7af-5ec5-4fb9-baa3-1a6c025ae810.png)

---

## MongoDB Storage Engines

### Storage Engine이란?
- 데이터가 메모리와 디스크에 어떻게 저장하고 읽을지 관리하는 컴포넌트이다
- MySQL과 동일하게 Plugin 형태로 되어 있어 MongoDB도 다양한 Storage Engine을 사용할 수 있다
- MongoDB 3.2부터 MongoDB의 기본 Storage Engine은 WiredTiger이다(기존에는 MMAPv1사용)
- WiredTiger가 도입되면서 MongoDB의 성능은 큰 폭으로 좋아졌다

### WiredTiger Sotrage Engine 개선 사항

||||
|:--:|:--|:--|
|항목|MMAPv1|WiredTiger|
|Data Compression|지원하지 않는다| 지원한다|
|Lock|버전에 따라 Database 혹은 <br/> Collection 레벨의 Lock| Document레벨의 Lock|