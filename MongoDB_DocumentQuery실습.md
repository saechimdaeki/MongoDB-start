# MongoDB Document Query

## MongoDB Query Language

```
sql : select * from table;
mql : db.collection.find({});
```

### [sql vs mql](https://www.mongodb.com/docs/manual/reference/sql-comparison/) 


## Query Filter And Operator


#### Create Operations
```mongo
db.users.insertOne(     <- collection
    {
        name: "sue",      <- field: value
        age: 26,          <- field: value
        status: "pending" <- field: value 
    }  <- document
)
```

#### Read Operations

```mongo
db.users.find(           <- collection
    { age: { $gt: 18 } }, <- query criteria
    { name: 1, address : 1 } <- projection
).limit(5)     <- cursor modifier
```

#### Update Operations

```mongo
db.users.updateMany(   <- collection
    { age: { $lt: 18 } }, <- update filter
    { $set: { status: "reject" } } <- update action
)
```

#### Delete Operations

```mongo
db.users.deleteMany(     <- collection
    { status: "reject" } <- delete filter
)
```

## 기본 CRUD

```shell
mongosh "mongodb+srv://cluster0.mubmocp.mongodb.net/myFirstDatabase" --apiVersion 1 --username mongodb_user
```

sample dataset인 test db사용

```shell
use test
```

employees에 사원을 한명 넣어보자

```mongo
db.employees.insertOne({
    name: "saechimdaeki",
    age: 29,
    dept: "BackEnd",
    joinDate: new ISODate("2023-01-01"),
    salary: 800000,
    bonus: null
})
```

<img width="479" alt="image" src="https://user-images.githubusercontent.com/40031858/210161758-3524f8e8-ab8b-4938-be36-79ebf6e1799b.png">


여러 document 넣기

```mongo
db.employees.insertMany([
    {
    name: "junseong",
    age: 30,
    dept: "DevOps",
    joinDate: new ISODate("2023-01-01"),
    salary: 1000000,
    resignationDate: new ISODate("2023-03-03"),
    bonus: null
    },
    {
    name: "intern",
    age: 35,
    dept: "DevOps tmp",
    isNegotiating: true
    }
])
```

<img width="469" alt="image" src="https://user-images.githubusercontent.com/40031858/210161801-0d273fd3-4050-43bc-89ff-634da71c35cf.png">


업데이트 쿼리

```mongo
db.employees.updateOne(
    { name : "intern" },
    {
        $set: {
            salary: 500000,
            dept: "DevOps",
            joinDate: new ISODate("2023-01-01"),
        },
        $unset:{
            isNegotiating:""
        }
    }
)
```

다음과 같이 변경 되었음

<img width="425" alt="image" src="https://user-images.githubusercontent.com/40031858/210161907-af8e9b2a-8d82-4eba-918d-cffe3acea51a.png">


현재 근무하고 있는 사원들에게 연봉 10%인상을 해보자

```
db.employees.updateMany(
    { resignationDate: { $exists: false }, joinDate: { $exists: true } },
    { $mul: { salary: Decimal128("1.1")} }
)

```

intern을 삭제해보자

```mongo
db.employees.deleteOne({name:"intern"})
```

<img width="541" alt="image" src="https://user-images.githubusercontent.com/40031858/210162022-f4c04c2b-eeac-46b5-8310-1060e25aaf8b.png">



조회용 쿼리 실습은 sample_guides db활용 

```mongo
db.planets.findOne({name: "Mars"})
```

조건 추가한 검색 (,는 기본적으로 and라고 보면 됨.)

```mongo
db.planets.find({
    hasRings: true, 
    orderFromSun: {$lte: 6}
}) 
```

and조건에 대한 Operator도 존재한다

```mongo
db.planets.find({
    $and: [
        { hasRings: true},
        { orderFromSun: { $lte: 6}}
    ]
})
```

```mongo
db.planets.find({
    $or: [
        { hasRings: {$ne: false} },
        { orderFromSun: { $gt: 6}}
    ]
})
```

배열안에 특정 값이 있으면 반환하게 끔도 가능.

```mongo
db.planets.find(
    { mainAtmosphere: {$in: ['O2']  }}
)
```

## 유용한 Query 함수들

https://www.mongodb.com/docs/v5.0/reference/method/

#### bulkWrite()
- 여러 종류의 변경사항을 배치성 작업으로 수행해야할 때 유용하게 사용할 수 있는 함수

```mongo
db.collection.bulkwrite(
    [ <operation 1>, <operation2>, ...],
    {
        writeConcern: <document>,
        ordered: <boolean>
    }
)
```

```mongo
db.bulk.bulkWrite(
    [
        {insertOne: {doc:1 , order:1}},
        {insertOne: {doc:2 , order:2}},
        {insertOne: {doc:3 , order:3}},
        {insertOne: {doc:4 , order:4}},
        {insertOne: {doc:4 , order:5}},
        {insertOne: {doc:5 , order:6}},
        {
            deleteOne: {
                filter: {doc: 3}
            }
        },
        {
            updateOne: {
                filter: {doc: 2},
                update: {
                    $set: {doc: 12}
                }
            }
        }
    ]
)
```

<img width="510" alt="image" src="https://user-images.githubusercontent.com/40031858/210190664-7116a3a3-9724-42be-b337-b68f474e0f19.png">



ordered false라면 순서와 상관없이 성능을 우선시 해서 함수를 실행할 수 있음.
```mongo
db.bulk.bulkWrite(
    [
        {insertOne: {doc:1 , order:1}},
        {insertOne: {doc:2 , order:2}},
        {insertOne: {doc:3 , order:3}},
        {insertOne: {doc:4 , order:4}},
        {insertOne: {doc:4 , order:5}},
        {insertOne: {doc:5 , order:6}},
        {
            updateOne: {
                filter: {doc: 2},
                update: {
                    $set: {doc: 12}
                }
            }
        },
        {
            deleteMany: {
                filter: {doc: 3}
            }
        },
    ],
    {ordered: false}
)
```

mongodb는 document 레벨에서 원자성을 보장. insertMany함수를 해서 100만개의 데이터를 삽입되고 있는데 중간에 문제가 발생되면

롤백하지 않고 삽입한 곳 까지만하고 명령을 끝내버림.

#### countDocuments()

- document에 대한 수를 확인 

```mongo
db.bulk.countDocuments()
```

#### distinct()
- 고유한 필드의 값을 가져옴

```mongo
db.bulk.distinct("doc")
```

<img width="588" alt="image" src="https://user-images.githubusercontent.com/40031858/210192227-8fce6fa3-f9a1-47a9-93d2-c7767d7f55f9.png">

#### findAndModify()

- 동시성에 대한 제어를 위한 함수. 수정하기 전에 document를 반환하고 수정하거나 수정하고 수정된 값을 반환

```mongo
db.bulk.findAndModify({
    query: { doc: 4 },
    update: { $inc: {doc: 1} }
})
```
<img width="931" alt="image" src="https://user-images.githubusercontent.com/40031858/210192302-76ea5b48-1fc9-4e46-8838-6bf0614df1f2.png">

#### getIndexes()

- 해당 collection에 생성되어 있는 index를 볼 수 있는 쿼리

<img width="559" alt="image" src="https://user-images.githubusercontent.com/40031858/210192465-02e8842c-835d-4b87-912f-3bdd0f2a2cd3.png">

#### replaceOne()

- document 전체를 수정할 때 사용하는 함수 _id는 변경되지 않음 

```mongo
db.bulk.replaceOne({ doc: 1}, {doc: 13})
```

<img width="597" alt="image" src="https://user-images.githubusercontent.com/40031858/210192627-9f57063d-b883-4e1a-b502-612cbe08adf2.png">

