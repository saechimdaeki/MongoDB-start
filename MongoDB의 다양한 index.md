# MongoDB의 다양한 Index

## MongoDB의 Index 기본 구조와 효율적인 탐색

<img width="1139" alt="image" src="https://user-images.githubusercontent.com/40031858/212530085-1c6a24d6-6c6f-4deb-b454-30df3971b321.png">

<img width="1117" alt="image" src="https://user-images.githubusercontent.com/40031858/212530269-8f028481-e9b0-4944-b310-eee29001fb14.png">

<img width="1335" alt="image" src="https://user-images.githubusercontent.com/40031858/212530340-f86ad9d8-8dfc-46ed-a754-44b979e507e4.png">

인덱스의 내부 구조는 다음과 같음

<img width="716" alt="image" src="https://user-images.githubusercontent.com/40031858/212530362-303fcb39-477c-4cfd-995a-8480be44d510.png">

## Compound Index와 ESR Rule

### Compound Index

여러 필드로 구성되어 있는 index. 일단 첫 번째로 인덱스의 선행 필드 일부만 사용해도 인덱스를 사용할 수 있다.

![image](https://user-images.githubusercontent.com/40031858/212530701-4b183e86-d99c-4df5-a91c-92a62e5bd8a6.png)

## E-S-R Rule

![image](https://user-images.githubusercontent.com/40031858/212530787-4835468b-47cc-40ca-8c7c-103f5fd758dc.png)


![image](https://user-images.githubusercontent.com/40031858/212531243-9bbeadaf-9b75-47a8-8aba-2ac50f14ed8d.png)


### E -> R Equality Before Range

<img width="1094" alt="image" src="https://user-images.githubusercontent.com/40031858/212531258-64db255d-4ed2-4058-9bbd-e52ce5af4b45.png">

### E -> S Equality Before Sort

<img width="1115" alt="image" src="https://user-images.githubusercontent.com/40031858/212531331-c3f4b760-ad39-45d3-ba2a-67a9697d5265.png">

###  E-S-R Equality Sort Range

<img width="1100" alt="image" src="https://user-images.githubusercontent.com/40031858/212531377-168d0fc7-33ff-4f46-bf57-a09ef971f59d.png">

#### 쿼리 연습 해보기

```mongo
use sample_training

show collections

db.zips.findOne()

db.zips.getIndexes()

db.zips.find(
	{
		state: "LA",
		pop: {
			$gte: 40000
		}
	}
).sort({city: 1})


db.zips.find(
	{
		state: "LA",
		pop: {
			$gte: 40000
		}
	}
).sort({city: 1}).explain('executionStats')

db.zips.createIndex({ state: 1 })

db.zips.getIndexes()

db.zips.find(
	{
		state: "LA",
		pop: {
			$gte: 40000
		}
	}
).sort({ city: 1 }).explain('executionStats')


db.zips.createIndex({ state: 1, city: 1, pop: 1 })

db.zips.getIndexes()


db.zips.find(
	{
		state: "LA",
		pop: {
			$gte: 40000
		}
	}
).sort({ city: 1 }).explain('executionStats')



db.zips.find(
	{
		state: "LA",
		pop: {
			$gte: 40000
		}
	},
	{
		_id: 0,
		state: 1,
		pop: 1,
		city: 1
	}
).sort({ city: 1 }).explain('executionStats')
```


---

## Multikey Index

몽고 DB는 배열필드거나 배열안에 내장되어 있는 필드로 index를 만들 수 있고 그렇게 생성하면 multikey index로 구분이된다.

<img width="960" alt="image" src="https://user-images.githubusercontent.com/40031858/212537927-550cdedc-8fe8-48e1-b145-87dc312e75da.png">


<img width="808" alt="image" src="https://user-images.githubusercontent.com/40031858/212537937-d7bfedfb-400f-42b3-a25c-04e7cc7e3f6c.png">

### Multikey Index의 비용은 다음과 같다

<img width="781" alt="image" src="https://user-images.githubusercontent.com/40031858/212537952-2553d904-93c9-4a64-9d28-63815f0c22bd.png">

---

## Index 생성시 주의사항 

#### Background Option

<img width="1163" alt="image" src="https://user-images.githubusercontent.com/40031858/212538155-f6ce8044-39d2-4f1b-911b-96122c9cbc4f.png">


구문검사가 취약한 면이 있다.

<img width="1044" alt="image" src="https://user-images.githubusercontent.com/40031858/212538235-110ea6bc-10f5-4f01-8b7f-d7fc83b7efb3.png">

4.4 이전 까지 Index는 내부적으로 Primary에서 생성 완료하고 Secondary에 복제한다.

Index 생성으로 인해서 발생하는 성능저하를 줄이기 위해 멤버 하나씩 접속해서 Rolling 형태로 Index를 생 성했다.
- 하지만 너무 번거롭다.

- Unique Index는 Collection에 대해서 Write가 없다는 것을 확인하고 생성해야한다. 
- Index 생성 시간이 Oplog Window Hour보다 작아야한다.

#### Drop Index

<img width="1097" alt="image" src="https://user-images.githubusercontent.com/40031858/212538289-cc361618-96d5-47fe-ae82-10b30e90e3c4.png">


버전 5.0 부터, Index 생성 중에 정상적으로 process가 shutdown되면 다시 기동 되었을 때 기존의 progress에 이어서 Index가 생성된다.

비정상적으로 shutdown된 경우는 처음부터 Index를 다시 생성한다.


<img width="1145" alt="image" src="https://user-images.githubusercontent.com/40031858/212538338-93dfe227-9f6b-4723-9641-55328c0cd689.png">