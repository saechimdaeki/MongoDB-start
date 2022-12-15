
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