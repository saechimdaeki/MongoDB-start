# MongoDB Aggregation


## 집계 프레임워크 Aggregation 소개

### Aggregation이란?
- Collection의 데이터를 변환하거나 분석하기 위해 사용하는 집계Framework
- Aggregation은 Find함수로 처리할 수 없는, SQL의 Group By와 Join구문 같은 복잡한 데이터 분석 기능들을 제공한다
- Aggregation Framework는 Pipeline형태를 갖춘다(Linux Pipeline 구문과 동일한 방식)
- MongoDB 2.2부터 제공되었고 이전에는 Map Reduce를 사용했다.


### Aggregation Example 

```sql
select 
    productName,
    SUM(quantity) AS sumQuantity
FROM orders
WHERE status = 'urgent'
GROUP BY productName;
```

```mql
db.orders.aggregate([
    {$match: {status: "urgent"},
    {$group: {
        _id: "$productName",
        sumQuantity:{$sum: "$quantity"}
    }}
])
```

<img width="963" alt="image" src="https://user-images.githubusercontent.com/40031858/210231235-6f453bda-f0ea-4990-a4bf-b137056fceed.png">


<img width="1267" alt="image" src="https://user-images.githubusercontent.com/40031858/210231275-7b15136c-2e50-49b4-bb1e-679071eeddae.png">

---

## 자주 사용되는 Aggregation Stage

일단 실습 진행에 앞서 다음과 같은 insert를 실행

```mongo
db.orders.insertMany( [
	{ _id: 0, name: "Pepperoni", size: "small", price: 19,
	  quantity: 10, date: ISODate( "2021-03-13T08:14:30Z" ) },
	{ _id: 1, name: "Pepperoni", size: "medium", price: 20,
	  quantity: 20, date : ISODate( "2021-03-13T09:13:24Z" ) },
	{ _id: 2, name: "Pepperoni", size: "large", price: 21,
	  quantity: 30, date : ISODate( "2021-03-17T09:22:12Z" ) },
	{ _id: 3, name: "Cheese", size: "small", price: 12,
	  quantity: 15, date : ISODate( "2021-03-13T11:21:39.736Z" ) },
	{ _id: 4, name: "Cheese", size: "medium", price: 13,
	  quantity:50, date : ISODate( "2022-01-12T21:23:13.331Z" ) },
	{ _id: 5, name: "Cheese", size: "large", price: 14,
	  quantity: 10, date : ISODate( "2022-01-12T05:08:13Z" ) },
	{ _id: 6, name: "Vegan", size: "small", price: 17,
	  quantity: 10, date : ISODate( "2021-01-13T05:08:13Z" ) },
	{ _id: 7, name: "Vegan", size: "medium", price: 18,
	  quantity: 10, date : ISODate( "2021-01-13T05:10:13Z" ) }
])
```

```mongo
db.orders.aggregate([
    {
        $match:{
            size: "medium"
        }
    },
    {
        $group: {
            _id: {$getField: "name"},
            totalQuantity: {
                $sum: {$getField: "quantity"}
            }
        }
    }
])
```

<img width="499" alt="image" src="https://user-images.githubusercontent.com/40031858/210231826-c0f31b79-6c77-4254-9f6e-af5633982f25.png">

위의 mql중 getField는 다음과 같이 생략하여 사용할수도 있다.


```mongo
db.orders.aggregate([
    {
        $match:{
            size: "medium"
        }
    },
    {
        $group: {
            _id: "$name",
            totalQuantity: {
                $sum:  "$quantity"
            }
        }
    }
])
```

이번에는 날짜가 2022년부터 2년간 판매된 데이터 중 날짜별 매출과 평균 판매수량을 출력하고 매출로 내림차순으로 나타내보자.

```mongo
db.orders.aggregate([
    {
        $match: {
            date: {
                $gte: new ISODate("2020-01-30"),
                $lt: new ISODate("2022-01-30")
            }
        }
    },
    {
        $group: {
            _id: {$dateToString: {
                format: "%Y-%m-%d", date: "$date"
                }
            },
            totalOrderValue: {
                $sum: {
                    $multiply: ["$price", "$quantity"]
                }
            },
            averageOrderQuantity: {
                $avg: "$quantity"
            }
        }
    },
    {
        $sort: {
            totalOrderValue: -1
        }
    }
])
```

<img width="602" alt="image" src="https://user-images.githubusercontent.com/40031858/210232448-b782d71d-e1b2-49d2-b100-dd6349475d53.png">


이번에는 배열로 데이터를 출력하기위해 다음과 같은 데이터를 넣어주자

```mongo
 db.books.insertMany([
	{ "_id" : 8751, "title" : "The Banquet", "author" : "Dante", "copies" : 2 },
	{ "_id" : 8752, "title" : "Divine Comedy", "author" : "Dante", "copies" : 1 },
	{ "_id" : 8645, "title" : "Eclogues", "author" : "Dante", "copies" : 2 },
	{ "_id" : 7000, "title" : "The Odyssey", "author" : "Homer", "copies" : 10 },
	{ "_id" : 7020, "title" : "Iliad", "author" : "Homer", "copies" : 10 }
 ])
```

저자를 중심으로 grouping한다음에 저자가 쓴 책을 배열로 출력해보자

```mongo
db.books.aggregate([
    {
        $group: {
            _id : "$author",
            books: {
                $push: "$title"
            }
        }
    }
])
```

<img width="454" alt="image" src="https://user-images.githubusercontent.com/40031858/210232663-21501c78-814f-424a-824b-02532f04d303.png">

필드에 대한 값이 아니라 document자체를 넣으려면 어떻게 해야할까? 다음과 같이 하면 된다.


```mongo
db.books.aggregate([
    {
        $group: {
            _id : "$author",
            books: {
                $push: "$$ROOT"
            },
            totalCopies: {
                $sum: "$copies"
            }
        }
    }
])
```


`$$ROOT`는 가장 top레벨 시스템 변수이다.

<img width="566" alt="image" src="https://user-images.githubusercontent.com/40031858/210232868-a38cd02d-bae2-4bea-bde0-088163c4f8a1.png">


따로 스테이지로 분리해서 다음과 같이 할 수도 있다

```mongo
db.books.aggregate([
    {
        $group: {
            _id : "$author",
            books: {
                $push: "$$ROOT"
            }
        }
    },
    {
        $addFields: {
            totalCopies : {$sum : "$books.copies"}
        }
    }
])
```


다음 실습을 위해 데이터를 넣어주자

```mongo
db.orders.drop()

db.orders.insertMany([
    { "productId" : 1,   "price" : 12,   },
    { "productId" : 2,   "price" : 20,   },
    { "productId" : 3,   "price" : 80,   }
])

 
db.products.insertMany([
    { "id" : 1,  "instock" : 120 },  
    { "id" : 2,  "instock" : 80  }, 
    { "id" : 3,  "instock" : 60  }, 
    { "id" : 4,  "instock" : 70  }
])
```

조인(lookup)은 이런 식으로 사용할 수 있다

```mongo
db.orders.aggregate([
    {
        $lookup: {
            from: 'products',
            localField: 'productId',
            foreignField: 'id',
            as: 'data'
        }
    }
])
```

같은 필드를 기준으로 비교할 때 $expr operator를 사용 할 수 있다. 

```mongo
db.orders.aggregate([
    {
        $lookup: {
            from: 'products',
            localField: 'productId',
            foreignField: 'id',
            as: 'data'
        }
    },
    {
        $match: {
            $expr: {
                $gt: ['$data.instock', '$price']
            }
        }
    }
])
```

다만 위와 같은 mql은 원하는 결과가 나오지 않는다. $expr은 배열을 이용하게되면 정상적인 결과를 얻을 수 없다.

$expr은 필드와 같은 타입은 사용할 수 있지만 . 형태로 배열안으로 들어가면 정확한 결과가 나오지않는다

이럴때는 $unwind operator를 사용하면 된다.


```mongo
db.orders.aggregate([
    {
        $lookup: {
            from: 'products',
            localField: 'productId',
            foreignField: 'id',
            as: 'data'
        }
    },
    {
        $unwind: '$data'
    },
    {
        $match: {
            $expr: {
                $gt: ['$data.instock', '$price']
            }
        }
    }
])
```


그럼 이렇게 하면 어떤 결과가 나올까

```mongo
db.books.aggregate([
	{
		$group: {
			_id: "$author",
			books: {
				$push: "$$ROOT"
			}
		}
	},
	{
		$addFields: {
			totalCopies : {$sum: "$books.copies"}
		}
	},
	{
		$unwind: '$books'
	}
])
```


<img width="818" alt="image" src="https://user-images.githubusercontent.com/40031858/210234117-5e53b128-efb8-4ff3-969c-d97612f25543.png">

unwind operator는 이처럼 경우에 따라서 grouping했는데 다시 풀어줘야 하는경우, join해서 배열이 생겼는데 이를 풀어줘야 하는 경우에 사용된다


paging은 limit와 skip을 사용할 수 있음

```mongo
db.listingsAndReviews.aggregate([
	{
		$match: {
			property_type: "Apartment"
		}
	},
	{
		$sort: {
			number_of_reviews: -1
		}
	},
	{
		$skip: 0
	},
	{
		$limit: 5
	},
	{
		$project: {
			name: 1,
			number_of_reviews: 1
		}
	}
])
```

다른 컬렉션에 저장하고 싶을때 사용하는 stage가 있다 . 바로 $out이라는 것이다. 

```mongo
db.books.aggregate([
	{
		$group: {
			_id: "$author",
			books: {$push: "$title"}
		}
	},
	{
		$out: "authors"
	}
])
```



<img width="706" alt="image" src="https://user-images.githubusercontent.com/40031858/210234943-ea87fdc4-cfcf-4914-9d82-938e1784c9ff.png">

authors라는 collection이 생성된 것을 볼 수 있다.




-----

# MongoDB Aggregation 예제

# MongoDB Aggregation 예제

#### sample_traning database의 grades collection을 이용하고 class_id별 exam과 quiz에 대한 평균값을 구한다. exam과 quiz의 평균값은 배열 필드안에 넣어서 저장한다.

##### result example 

```mongo
[
  {
    _id: 500,
    scores: [
      { k: 'exam', v: 45.04082924638036 },
      { k: 'quiz', v: 51.35603805996617 }
    ]
  },
  {
    _id: 499,
    scores: [
      { k: 'exam', v: 52.234488613263466 },
      { k: 'quiz', v: 48.96268506750967 }
    ]
  },
  {
    _id: 498,
    scores: [
      { k: 'exam', v: 48.51775335555769 },
      { k: 'quiz', v: 53.827492248151465 }
    ]
  },
  {
    _id: 497,
    scores: [
      { k: 'exam', v: 50.80561533355925 },
      { k: 'quiz', v: 51.27682967858154 }
    ]
  },
  {
    _id: 496,
    scores: [
      { k: 'exam', v: 47.28546854417578 },
      { k: 'quiz', v: 50.30975687853305 }
    ]
  }
]
```

#### solution1 2개 필드를 grouping하고 다시 grouping해서 원하는 결과를 얻는 방법

```mongo
db.grades.aggregate([
	{
		$unwind: "$scores"
	},
	{
		$match: {
			"scores.type": {
				$in: ['exam', 'quiz']
			}
		}
	},
	{
		$group: {
			_id: {
				class_id: "$class_id",
				type: "$scores.type"
			},
			avg_score: {
				$avg: "$scores.score"
			}
		}
	},
	{
		$group: {
			_id: "$_id.class_id",
			scores: {
				$push: {
					type: "$_id.type",
					avg_score: "$avg_score"
				}
			}
		}
	},
	{
		$sort: {
			_id: -1
		}
	},
	{
		$limit: 5
	}
])
```

#### solution2 미리 원하는 조건으로 배열을 필터링하고 배열필드에 score에 대한 모든 값을 넣고 나중에 평균을 구하는 방법

```mongo
db.grades.aggregate([
	{
		$addFields: {
			tmp_scores: {
				$filter: {
					input: "$scores",
					as: "scores_var",
					cond: {
						$or: [
							{$eq: ["$$scores_var.type", 'exam']},
							{$eq: ["$$scores_var.type", 'quiz']}
						]
					}
				}
			}
		}
	},
	{
		$unset: ["scores", "student_id"]
	},
	{
		$unwind: "$tmp_scores"
	},
	{
		$group: {
			_id: "$class_id",
			exam_scores: {
				$push: {
					$cond: {
						if: {
							$eq: ["$tmp_scores.type", 'exam']
						},
						then: "$tmp_scores.score",
						else: "$$REMOVE"
					}
				}
			},
			quiz_scores: {
				$push: {
					$cond: {
						if: {
							$eq: ["$tmp_scores.type", 'quiz']
						},
						then: "$tmp_scores.score",
						else: "$$REMOVE"
					}
				}
			}
		}
	},
	{
		$project: {
			_id: 1,
			scores: {
				$objectToArray: {
					exam: {
						$avg: "$exam_scores"
					},
					quiz: {
						$avg: "$quiz_scores"
					}
				}
			}
		}
	},
	{
		$sort: {
			_id: -1
		}
	},
	{
		$limit: 5
	}
])
```