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
