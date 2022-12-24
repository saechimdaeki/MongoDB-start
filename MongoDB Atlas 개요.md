# MongoDB Atlas 개요


실습에 있어 무료버젼을 사용할 것이라서 다음과 같이 세팅함. (참고로 무료버젼의 경우 replica set형태의 배포만 가능)

<img width="895" alt="image" src="https://user-images.githubusercontent.com/40031858/208892906-36a52878-219d-47b7-b8fb-262a61df95a4.png">

실습용 환경이기 때문에 다음과 같이 Ip 주소를 할당

<img width="855" alt="image" src="https://user-images.githubusercontent.com/40031858/208893763-382e2f51-ce28-4fa7-9cc4-9a62affcb92e.png">


### MongoDB Shell로 접속하는 방법
```
brew install mongosh
```

```
mongosh "mongodb+srv://cluster0.mubmocp.mongodb.net/myFirstDatabase" --apiVersion 1 --username mongodb_user
```

<img width="1069" alt="image" src="https://user-images.githubusercontent.com/40031858/208894491-2d61dbb4-7f7f-4c37-82c9-315473e8d215.png">

---

# 번외1 : Replica Set 직접 구현

### Binary 설치

https://www.mongodb.com/try/download/community

### 환경 준비

```
mkdir -p mongodb/data{1,2,3}
mkdir -p mongodb/config
mkdir -p mongodb/logs
```

### 실행 방법1. Binary 옵션 이용

```markdown
cd ~/Downloads
cd mongodb-macos-x86_64-5.0.12
cd bin

# 각각 다른 터미널에서 실행한다.
mongod --replSet rs1 --port 27017 --bind_ip "0.0.0.0" --dbpath /Users/user/mongodb/data1  --oplogSize 128
mongod --replSet rs1 --port 27018 --bind_ip "0.0.0.0" --dbpath /Users/user/mongodb/data2  --oplogSize 128
mongod --replSet rs1 --port 27019 --bind_ip "0.0.0.0" --dbpath /Users/user/mongodb/data3  --oplogSize 128

```

### 실행 방법2. Config File이용
```
vim mongodb/config/mongod1.conf
vim mongodb/config/mongod2.conf
vim mongodb/config/mongod3.conf
```

#### mongod1.conf
```properties
net:
    port: 27017
    bindIp: 0.0.0.0

storage:
    dbPath: "/Users/user/mongodb/data1"
    directoryPerDB: true

replication:
    oplogSizeMB: 128
    replSetName: "rs1"

systemLog:
    path: "/Users/user/mongodb/logs/mongod1.log"
    destination: "file"
```

#### mongod2.conf
```properties
net:
    port: 27018
    bindIp: 0.0.0.0

storage:
    dbPath: "/Users/user/mongodb/data2"
    directoryPerDB: true

replication:
    oplogSizeMB: 128
    replSetName: "rs1"

systemLog:
    path: "/Users/user/mongodb/logs/mongod2.log"
    destination: "file"
```

#### mongod3.conf
```properties
net:
    port: 27019
    bindIp: 0.0.0.0

storage:
    dbPath: "/Users/user/mongodb/data3"
    directoryPerDB: true

replication:
    oplogSizeMB: 128
    replSetName: "rs1"

systemLog:
    path: "/Users/user/mongodb/logs/mongod3.log"
    destination: "file"
```

```
cd ~/Downloads
cd mongodb-macos-x86_64-5.0.12
cd bin

# 각각 다른 터미널에서 실행한다.
./mongod -f /Users/user/mongodb/config/mongod1.conf
./mongod -f /Users/user/mongodb/config/mongod2.conf
./mongod -f /Users/user/mongodb/config/mongod3.conf
```

## Replica Set Initiate

#### 멤버 접속

```
cd ~/Downloads
cd mongodb-macos-x86_64-5.0.12
cd bin

./mongosh "mongodb://localhost:27017"
```

#### Replica Set Initiate and Check
```
rs.initiate({
    _id: "rs1",
    members: [
        { _id: 0, host: "localhost:27017" },
        { _id: 1, host: "localhost:27018" },
        { _id: 2, host: "localhost:27019" },
    ],
});

rs.status;
```

---

# 번외2 : Sharded Cluster 직접 구현

도커로 진행할 것이기 때문에 다음 레포지토리로 세팅.. https://github.com/minhhungit/mongodb-cluster-docker-compose 

```
docker-compose exec configsvr01 sh -c "mongo < /scripts/init-configserver.js"
docker-compose exec configsvr02 sh -c "mongo < /scripts/init-configserver.js"
docker-compose exec configsvr03 sh -c "mongo < /scripts/init-configserver.js"
```

그 후 shard에 추가를 하자.

```
sh.addShard("rs-shard-01/shard01-a:27017")
sh.addShard("rs-shard-01/shard01-b:27017")
sh.addShard("rs-shard-01/shard01-c:27017")
sh.addShard("rs-shard-02/shard02-a:27017")
sh.addShard("rs-shard-02/shard02-b:27017")
sh.addShard("rs-shard-02/shard02-c:27017")
sh.addShard("rs-shard-03/shard03-a:27017")
sh.addShard("rs-shard-03/shard03-b:27017")
sh.addShard("rs-shard-03/shard03-c:27017")
```

---

## MongoDB에 연결하는 방식 소개

MongoDb는 두 가지의 Connection String이 존재한다

1. IP 리스트를 나열하는 `Standard Connection String Format`
2. IP에 대한 시드를 받아서 하나의 엔드포인트를 제공하는 `DNS Seed List Connection Format`

---

## MongoDB의 다양한 Client

1. Sheel을 이용하는 방법
```
mongosh "mongodb+srv://cluster0.mubmocp.mongodb.net/myFirstDatabase" --apiVersion 1 --username mongodb_user
```

2. MongoDB Compose를 사용하는 방법

<img width="1401" alt="image" src="https://user-images.githubusercontent.com/40031858/209417746-f35a7b8a-68a9-4423-b5e7-98d452636c90.png">

```
authentication에서 비밀번호를 세팅하자
```
<img width="1145" alt="image" src="https://user-images.githubusercontent.com/40031858/209417792-438d84d2-6b6b-4ef3-ae59-2d827231f29c.png">

쉽게 document를 찾을 수도 있음.


3. VSCode를 이용해서 접속하는 방법

    <img width="368" alt="image" src="https://user-images.githubusercontent.com/40031858/209417832-a5d9d19d-da2b-4a34-9ec5-ec705374483d.png">

    확장 프로그램 설치는 필수

    <img width="957" alt="image" src="https://user-images.githubusercontent.com/40031858/209417894-94b066a2-b37a-4f0d-b74e-efdce60cbfb2.png">

4. application을 통해 접속하는 방법

예시는 python으로 할 것.

```python
// 당연히 비밀번호는 변경해야함.
conn = "mongodb+srv://mongodb_user:<password>@cluster0.mubmocp.mongodb.net/?retryWrites=true&w=majority"
client = pymongo.MongoClient(conn, tlsCAFile = certifi.where())
db = client.word
db.abc.insert_one({"abc": 1})
print(db.abc.find_one({"abc":1}))
```

## MongoDB 인증과 권한

https://www.mongodb.com/docs/manual/reference/built-in-roles/

`db.serverStatus()` 명령어로 서버 상태를 확인하고 싶으나 다음과 같이 권한이 없다.

<img width="564" alt="image" src="https://user-images.githubusercontent.com/40031858/209418050-c75ed0a9-d641-4353-926a-0c3f0d43bd62.png">

공싟 문서를 보면 `clusterMonitor`라는 built-in Roles가 필요한상황.

<img width="949" alt="image" src="https://user-images.githubusercontent.com/40031858/209418149-9f600b26-b412-4aaa-af0a-9f127201dbe4.png">

monitor라는 6 hour temporary user를 만들은 후 위와 같이 권한을 주었다.

<img width="869" alt="image" src="https://user-images.githubusercontent.com/40031858/209418219-a72f4ba5-4232-4ca6-9ce7-dfa431c3ce22.png">

권한을 주니 success