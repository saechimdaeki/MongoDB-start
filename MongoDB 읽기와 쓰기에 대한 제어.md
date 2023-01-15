# 읽기와 쓰기에 대한 제어 

몽고 db의 일관성과 관련된 계층을 나누면 다음과 같이 이루어진다

<img width="1244" alt="image" src="https://user-images.githubusercontent.com/40031858/212520769-5eaebd16-2ff9-4f03-b6dd-837245d8471c.png">


mongodb는 기본적으로 Single Document에 대해서는 원자성을 보장하고 복제를 통해 ha를 구성하기 때문에 `언젠가는 한 시점에 일관적 데이터를 보장함` 

그 다음 multidocument에 대해서, 그리고 여러작업에 대한 원자성을 보장해주는 트랜잭션이 존재한다. mongodb에서 권장되진 않지만 기능적으로는 존재한다

동일한 데이터를 여러 멤버에 저장하는 replica set을 통해 ha를 구성하고 멤버간의 데이터 일관성 제어가 필요하다

마지막으로 sharded cluster에서 데이터 분산이 되어 샤드간 동일한 데이터를 갖지 않도록 제어가 필요 

우리가 제어할 수 있는 부분은 replica set, sharded cluster라고 보면 된다.


### Replica set

<img width="1081" alt="image" src="https://user-images.githubusercontent.com/40031858/212520875-eb4bf590-476d-4b5a-90af-e6d4a3b00c9c.png">


---

## Read Preference

<img width="992" alt="image" src="https://user-images.githubusercontent.com/40031858/212522133-4e3b6cae-1e1f-4d25-b530-0194ee5dd763.png">

<img width="1253" alt="image" src="https://user-images.githubusercontent.com/40031858/212522211-ab6f2306-26ef-433b-b218-435a43f4d3e1.png">


### 요청을 ReadPreference주고 해보기 (python)

```python
from pymongo import MongoClient
form pymong.read_Preferences import ReadPreference
import certifil

conn = "mongodb+srv://mongodb_user:940215@cluster0.mubmocp.mongodb.net/?readPreference=secondary"
client = MongoClient(conn, tlsCAFile = certifi.where())

db = client.test

db.fruits.insert_many([
    {
        "name" : "melon",
        "qty": 1000,
        "price": 16000
    },
    {
        "name" : "starwberry",
        "qty": 1000,
        "price": 10000
    },
    {
        "name" : "grape",
        "qty": 1500,
        "price": 5000
    }    
])

query_filter = {"name": "melon"}

while True:
    res = db.fruits.find_one(query_filter)
    print(res)
```

---

## Read_Write Concern

#### Write Concern

<img width="908" alt="image" src="https://user-images.githubusercontent.com/40031858/212522585-a535c612-4ec2-4a9e-a8bc-2ff76c7bf890.png">

write concern의 주 목적은 rollback을 방지하는 것이다.


<img width="845" alt="image" src="https://user-images.githubusercontent.com/40031858/212522691-dafdc935-e262-4753-b641-b8ced05084b7.png">

현재 A멤버가 primary인상태. c,d,e는 복제를 하는 중인 상태이다 (oplog)

datacenter1에서 네트워크 문제가 발생하게 된다면?

<img width="838" alt="image" src="https://user-images.githubusercontent.com/40031858/212522712-6f789a56-faf4-43a0-af12-48f6ef0103b6.png">

1번 데이터센터에서는 레플리카 셋 5개 중 2개만 통신이 되는 상황이니 과반수가 넘지 못해 A는 secondary로 내려오고 primary를 선출하지 못한다

2번 데이터 센터는 세개를 가지고 있어서 1번과 통신이 안된다 하더라도 새로운 primary를 선출하게되고 c가 선출이 된 상태이다.

문제는 c는 125번 상태까지 복사한 상태에서 primary가 된것이다 (126~128 누락)


<img width="830" alt="image" src="https://user-images.githubusercontent.com/40031858/212522738-8ddd1f15-1582-4b17-a30b-b6246b5a31d4.png">


멤버 A는 primary인 c에 128부터 받아오려고 복제작업을 하려고 하는데 126~128이 없기 때문에 rollback을 하게 된다.

그 다음 새로운 126부터 131까지 복제를 하게되는 것이다.

### Read Concern

<img width="1255" alt="image" src="https://user-images.githubusercontent.com/40031858/212522823-f92fe986-c1ea-4ecf-b35c-4f12291fd5f2.png">

---

## Causal Consistency

<img width="1313" alt="image" src="https://user-images.githubusercontent.com/40031858/212523004-8ee400ee-cdce-493a-9ec8-bc921713171b.png">

###  Linearizable Read Concern vs Causal Consistency

- 3.6 버전 이전에서는 Read Your Own Writes를 보장하기 위해 Write Concern “majority”와 Read Concern “linearizable”을 사용했다.
- Causal Consistency는 하나의 Thread에서 작업하고 Linearizable Read Concern은 다른 Session의 여러 Thread의 작업을 적용하여 기다린다.
- Linearizable Read Concern은 Primary Read에 대해서만 가능하고 Causal Consistency에서는 사용하지 못한다.

### example

<img width="1211" alt="image" src="https://user-images.githubusercontent.com/40031858/212523021-e165cb5a-3a21-41e8-b428-d2ca6d328e15.png">

<img width="1213" alt="image" src="https://user-images.githubusercontent.com/40031858/212523062-55c7736e-1e8d-4d01-a8a3-cdb33b75044f.png">

<img width="1174" alt="image" src="https://user-images.githubusercontent.com/40031858/212523078-127328f5-79b5-4c05-9b06-7e50414b95f8.png">

---

## MongoDB Transaction

#### 번외로 사용하지 않는 것을 추천하는 기능임.


### Transaction을 사용하는 이유
여러 작업에 대해서 하나의 논리적인 단위로 취급하여 원자성을 보장하기 위해 사용
• ex1) 은행 송금 서비스
• ex2) 예약 시스템

### 일반적 transaction 구조

<img width="1051" alt="image" src="https://user-images.githubusercontent.com/40031858/212523190-615b87ff-9c6c-4f58-a131-76acb9b44795.png">

### MongoDB 4.0 Transaction

<img width="810" alt="image" src="https://user-images.githubusercontent.com/40031858/212523194-a4af8d31-17f3-49b2-b94a-adfca951148c.png">

