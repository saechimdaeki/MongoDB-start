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

