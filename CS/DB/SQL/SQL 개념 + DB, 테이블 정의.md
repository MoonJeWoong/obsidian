
Structured Query Language의 약자로, RDBMS를 다룰 때 사용되는 표준 언어이다.

SQL은 크게 3가지 부분으로 나뉜다. : DDL + DCL + VDL

### Database VS Schema

- Mysql에서는 Database와 Schema가 같은 뜻을 의미하지만 postgreSQL 에서는 schema가 database의 namespace를 의미한다.


### Attribute data 타입 정리

![](https://i.imgur.com/RprIW3o.png)

=> Decimal 과 Numeric 타입은 mySQL에서는 큰 차이가 없으나 SQL 표준 스펙 정의에 의하면, 주어진 수가 미리 정해진 자릿수를 넘기는 경우 Numeric은 넘어간 자릿수를 저장하지 않고 Decimal은 그래도 저장해준다는 차이가 존재한다.

![](https://i.imgur.com/QPqqbGe.png)


![](https://i.imgur.com/q6bXFFS.png)


![](https://i.imgur.com/X4PQMlI.png)



### Key Constraints

- Primary Key

![](https://i.imgur.com/yRURoiR.png)

위 Player 테이블 예시의 경우 2개 이상의 속성 조합이 Primary Key로 정의되어 있다. 그런데 중복되는 PK를 가지는 튜플이 2개이고, Primary Key를 구성하는 속성에 Null이 섞여 있다. 이러한 경우는 PK로 정의될 수 없다.

![](https://i.imgur.com/GW5sy0r.png)


- Unique
![](https://i.imgur.com/sr2lftk.png)
![](https://i.imgur.com/SXAVWyq.png)

![](https://i.imgur.com/7RyhkFp.png)


- Foreign Key

![](https://i.imgur.com/JJdrJvo.png)

![](https://i.imgur.com/nQOcHn7.png)

외래키 제약조건을 걸었을 때, 어떤 테이블의 속성을 참조하는지, 그리고 참조 값이 변경되거나 삭제되었을 때 어떻게 처리할 것인지를 명시할 수 있다.

Mysql에서는 Restrict와 No Action 옵션이 동일하게 적용되나 SQL 표준 스펙에 의하면 약간 다르다. Restrict는 아예 참조값의 변경이나 제거를 막는 것이고, No Action은 트랜잭션 단위 안에서는 변경을 허용하나, 트랜잭션 종료 이후 커밋 시에도 제약조건을 위배한다면 변경사항을 반영하지 못하도록 하는 옵션이다.

Mysql에서는 set default 옵션을 제대로 지원하지는 않는다.





### 테이블 정의 속성

- Default 옵션

![](https://i.imgur.com/2Srbq9U.png)


- Check 옵션

![](https://i.imgur.com/5OgLpVb.png)

![](https://i.imgur.com/s4ztzbE.png)



### 테이블 생성 이후 스키마 변경하기 (ALTER)

![](https://i.imgur.com/xRDvFVP.png)


![](https://i.imgur.com/LZ80nSm.png)















---

[쉬운 코드 강의 1](https://www.youtube.com/watch?v=c8WNbcxkRhY&list=PLcXyemr8ZeoREWGhhZi5FZs6cvymjIBVe&index=3)