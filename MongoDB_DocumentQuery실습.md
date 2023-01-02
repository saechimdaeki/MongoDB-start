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


---

## 배열과 내장 Document를 다루는 방법

실습은 sample data set인 sample_supplies사용

```
use sample_supplies
show collections
-- sales
```

내장 document에 대한 field로 조회를 하려면 내장 document 전체를 그대로 사용해야 조회 가능

```mongo
db.sales.findOne({
    customer: { gender: 'F', age: 39, email: 'beecho@wic.be', satisfaction: 3 }
})
```

내장 document안에 있는 필드로 조회할 때는 다음과 같이 가능하다

```mongo
db.sales.findOne({
    "customer.email": "keecade@hem.uy"
})


db.sales.findOne({
    "customer.age": {$lt: 20}
})
```

```mmongo
db.inventory.insertMany([
   { item: "journal", qty: 25, tags: ["blank", "red"], dim_cm: [ 14, 21 ] },
   { item: "notebook", qty: 50, tags: ["red", "blank"], dim_cm: [ 14, 21 ] },
   { item: "paper", qty: 100, tags: ["red", "blank", "plain"], dim_cm: [ 14, 21 ] },
   { item: "planner", qty: 75, tags: ["blank", "red"], dim_cm: [ 22.85, 30 ] },
   { item: "postcard", qty: 45, tags: ["blue"], dim_cm: [ 10, 15.25 ] },
   { item: "postcard", qty: 45, tags: ["blue", "red"], dim_cm: [ 13, 14 ] }
]);
```


배열도 마찬가지로 field그대로 queryfilter를 사용하면 내장 document처럼 순서까지 동일해야 결과가 나옴.

```mongo
db.inventory.find({
    tags:['red', 'blank']
})
```

순서가 변경이 되어도 'red'와 'blank'가 모두 들어간 값들을 검색하려면 다음과 같이 할 수 있다

```mongo
db.inventory.find({
    tags: { $all: ['red', 'blank'] }
})
```

'red'나 'blank'가 들어간 값들을 검색하면 다음과 같이

```mongo
db.inventory.find({
    tags: { $in: ['red', 'blank'] }
})
```

배열 요소 중에 하나라도 queryfilter를 만족하는 값을 찾을 때 다음과 같이 할 수도 있다

```mongo
db.inventory.find({
    tags: 'blue'
})
```

배열 요소 중 하나라도 조건에 모두 만족하는 요소가 있다면 반환해주는 operator예시는 다음과 같다

```mongo
db.inventory.find({
    dim_cm: {$elemMatch: {$gt: 15, $lt: 20}}
})
```

배열의 특정 위치에 조건절을 주고 싶다면 다음과 같이 할 수 있다

```mongo
db.inventory.find({
    "dim_cm.1": {$lt: 20}
})
```


```mongo
db.sales.find({
    "items.name": 'binder',
    "items.quantity": {$lte: 6}
})
```

포지셔닝 오퍼레이터를 사용하면 조건에 만족하는 첫번째 요소만 반환할 수 있음.

```mongo
db.sales.find(
    {
        items: {
           $elemMatch: {
                name: "binder",
                quantity: {$lte: 6}
            }
        }
    },
    {
        saleDate: 1,
        "items.$": 1,
        storeLocation: 1,
        customer: 1
    }
)
```

<img width="939" alt="image" src="https://user-images.githubusercontent.com/40031858/210196365-c32eccf8-1779-4bbb-851a-3be5f57e1457.png">


---

다시 test로 넘어와서 다음과 같은 데이터를 삽입

```mongo
db.students.insertMany([
    {_id: 1, grades: [85, 80, 80]},
    {_id: 2, grades: [88, 90, 92]},
    {_id: 3, grades: [85, 100, 90]},
])
```

배열 요소 중 grade가 80인 첫 번째 요소를 82로 변경해보자

```mongo
db.students.updateOne(
    {_id: 1, grades: 80},
    {$set: {"grades.$": 82}}
)
```

<img width="532" alt="image" src="https://user-images.githubusercontent.com/40031858/210196560-fc8c18a7-20be-4a51-b576-7f6199949d99.png">


모든 document에 대해 모든 grade를 10씩 올리려면 다음과 같이 할 수 있다

```mongo
db.students.updateMany(
    {},
    {$inc: {"grades.$[]": 10}}
)
```

다음 실습을 위해 데이터를 더 넣어주자

```mongo
db.students.insertMany([
    {
        _id: 4,
        grades: [
            {grade: 80, mean: 75, std: 8},
            {grade: 85, mean: 90, std: 5},
            {grade: 85, mean: 85, std: 8},
        ]
    }
])
```


처음 만나는 grade가 85이상인 값의 std를 6으로 변경해보자

```mongo
db.students.updateOne(
    { _id: 4, "grades.grade": 85},
    {$set: {"grades.$.std": 6}}
)
```

마찬가지로 elemMatch도 사용할 수 있다

```mongo
db.students.updateOne(
    { _id: 4, grades: {$elemMatch: {grade: {$gte: 85}}}},
    {$set: {"grades.$.grade": 100}}
)
```

해당 document에 대해 grade가 87이상인 배열 요소에만 대해서 grade를 100으로 변경해보자

```mongo
db.students.updateMany(
    { _id: 6},
    { $set: { "grades.$[element].grade": 100 }},
    { arrayFilters: [{"element.grade": {$gte: 87}}]}
)
```

```mongo
db.students.updateOne(
    { _id: 7 },
    {$inc: {"grades.$[].questions.$[score]": 2}},
    {arrayFilters: [{score: {$gte: 8}}]}
)
```

이제 배열의 값을 넣고 빼보자

```mongo
db.shopping.insertMany([
    {
        _id: 1,
        cart: ['banana', 'cheeze', 'milk'],
        coupons: ['10%', '20%', '30%']
    },
    {
        _id: 2,
        cart: [],
        coupons: []
    }
])
```

배열의 값이 없는 경우에만 삽입하는 operatordls set을 사용해보자

```mongo
db.shopping.updateOne(
    { _id: 1 },
    {$addToSet: {cart: 'beer' }}
)
```

<img width="854" alt="image" src="https://user-images.githubusercontent.com/40031858/210197426-ebc4e08e-a24d-4d38-b516-a275d34c8bca.png">

수정이 된 건 0건이다 라는걸 볼 수 있음.


그렇다면 배열을 넣고 싶을때 이렇게 하면되지 않을까 생각할 수 있다

```mongo
db.shopping.updateOne(
    { _id: 1 },
    {$addToSet: {cart: ['beer', 'candy'] }}
)
```

<img width="554" alt="image" src="https://user-images.githubusercontent.com/40031858/210197580-338d2ddc-b438-443b-b023-9d835e0b64da.png">

하지만 기대와 달리 배열 자체가 들어가버린다.

이럴 때 우리는 each라는 것을 사용할 수 있다

```mongo
db.shopping.updateOne(
    { _id: 1 },
    {$addToSet: {cart: { $each: ['beer', 'candy']  }}}
)
```


<img width="461" alt="image" src="https://user-images.githubusercontent.com/40031858/210197613-bc2ce588-0524-46d0-87b8-58f3c782c8ae.png">


값을 기준으로 배열에서 제거하는 pull 이라는 operator도 있다

```Mongo
db.shopping.updateOne(
    { _id: 1 },
    {$pull: {cart: 'beer' }}
)
```

배열 양쪽 끝을 제어할 수 있는 pop과 push도 있다

```mongo
db.shopping.updateOne(
    {_id: 1},
    {$pop: {cart: 1, coupons: -1}}
)
```

push는 마지막에만 데이터를 넣어준다 


```mongo
db.shopping.updateOne(
    {_id: 1},
    {$push : {cart: 'popcorn'}}
)
```


position을 통해 어디에 넣을지 또한 가능하다.


```mongo
db.shopping.updateMany(
    {},
    {
        $push: {
            coupons:{
                $each: ['90%', '70%'],
                $position: 0
            }
        }
    }
)
```

slice라는 operator가 있는데 배열의 크기를 제한할 수 있는 operator이다.


```mongo
db.shopping.updateMany(
    {},
    {
        $push: {
            coupons:{
                $each: ['15%', '20%'],
                $position: 0,
                $slice: 5
            }
        }
    }
)
```