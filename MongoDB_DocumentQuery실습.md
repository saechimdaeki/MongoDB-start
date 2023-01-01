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