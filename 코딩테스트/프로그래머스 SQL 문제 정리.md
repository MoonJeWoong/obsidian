# SELECT

## 12세 이하인 여자 환자 목록 출력하기

``` sql
SELECT PT_NAME, PT_NO, GEND_CD, AGE, IFNULL(TLNO, 'NONE') AS TLNO
FROM PATIENT 
WHERE AGE <= 12 AND GEND_CD = 'W' 
ORDER BY AGE DESC, PT_NAME ASC;
```

- null인 값을 다른 값으로 변경해야 할 때 ifnull 메서드를 사용한다.
- order by를 사용할 때 먼저 정렬시키고픈 컬럼 순서대로 정의한다.


### 평균 일일 대여 요금 구하기

``` SQL
SELECT ROUND(SUM(DAILY_FEE) / COUNT(CAR_ID), 0) AS AVERAGE_FEE
FROM CAR_RENTAL_COMPANY_CAR
WHERE CAR_TYPE = 'SUV';
```

- 숫자 사칙연산은 원래 사용하던 대로 +, -, \*,  /, % 연산들을 사용해주면 된다.
- 소수점 반올림 
	- ROUND(NUM, 0) => 소수점 첫째 자리에서 반올림해서 정수를 만들어줌
	- ROUND(NUM, 1) => 소수점 첫째 자리부터 절삭, 아래에서 반올림
- 정수 올림 내림
	- CEILING(NUM) => 올림
	- FLOOR(NUM) =>  내림



# JOIN

### 조건에 맞는 도서와 저자 리스트 출력하기

``` SQL
SELECT BOOK_ID, AUTHOR_NAME, DATE_FORMAT(PUBLISHED_DATE, '%Y-%m-%d') AS PUBLISHED_DATE
FROM BOOK INNER JOIN AUTHOR ON BOOK.AUTHOR_ID = AUTHOR.AUTHOR_ID
WHERE BOOK.CATEGORY = '경제'
ORDER BY PUBLISHED_DATE;
```

- 날짜 형식을 지정해야 할 때는 DATE_FORMAT 메서드를 사용한다.
- A INNER JOIN B ON A.id = B.id 와 같은 형식으로 조인 시 매핑할 컬럼을 지정할 수 있다.
	- ON 절 사용하는 방법을 잘 숙지하자.


### 상품 별 오프라인 매출 구하기

``` SQL
SELECT PRODUCT_CODE, SUM(PRICE * SALES_AMOUNT) AS SALES
FROM PRODUCT P INNER JOIN OFFLINE_SALE O ON P.PRODUCT_ID = O.PRODUCT_ID
GROUP BY P.PRODUCT_CODE
ORDER BY SALES DESC, PRODUCT_CODE ASC;
```

- GROUP BY를 사용하지 않고 SUM() 메서드를 사용하면 SELECT에 명시된 다른 조회 컬럼은 무시되고 결과가 표시되므로 잊지 말고 GROUP BY를 사용해주자...!


### 없어진 기록 찾기

```SQL
SELECT O.ANIMAL_ID, O.NAME
FROM ANIMAL_OUTS O LEFT JOIN ANIMAL_INS I ON O.ANIMAL_ID = I.ANIMAL_ID
WHERE I.ANIMAL_ID IS NULL;
```

- 외부 결합 조인 수행 시 매핑되는 행이 없는 경우 해당 행의 컬럼들은 NULL 로 처리된다.
- 이 때 조인 시 사용한 테이블 명의 컬럼이 IS NULL인지 IS NOT NULL 인지를 이용해서 조인이 정상 처리되지 않은 레코드를 찾아낼 수 있다.


### 있었는데요 없었습니다

``` SQL
SELECT O.ANIMAL_ID, O.NAME
FROM ANIMAL_OUTS O LEFT JOIN ANIMAL_INS I ON O.ANIMAL_ID = I.ANIMAL_ID
WHERE O.DATETIME < I.DATETIME
ORDER BY I.DATETIME;
```

- DATETIME 도 대소 비교에서 작은 것이 더 앞선 시간대를 나타냄을 잊지말자.
- DATE_ADD, DATE_SUB 메서드와 INTERVAL 키워드를 통해 날짜를 더하거나 뺄 수 있다.

``` SQL
SELECT DATE_ADD(NOW(), INTERVAL 5 DAY);
SELECT DATE_ADD(NOW(), INTERVAL 3 HOUR);

SELECT DATE_SUB(NOW(), INTERVAL 5 DAY);
SELECT DATE_SUB(NOW(), INTERVAL 3 HOUR)
```


### 주문량이 많은 아이스크림들 조회하기

UNION ALL을 사용한 해결 방식
``` SQL
SELECT FLAVOR
FROM (SELECT * FROM FIRST_HALF
	 UNION ALL
	 SELECT * FROM JULY) AS HALFANDJULY
 GROUP BY FLAVOR
 ORDER BY SUM(TOTAL_PRICE)
 LIMIT 3;
```


- 해당 문제가 JOIN으로 분류되어 있기는 하나 실제로는 UNION ALL로 풀어내야 하는 문제임
- UNION을 사용하면 중복되는 ROW가 DISTINCT되는 로직이 수행되므로 UNION ALL을 사용해줘야 함
- FROM 에서 서브쿼리를 사용하게 되면 별칭을 꼭 붙여줘야 문법 에러가 발생하지 않는다.
- 상위 N 개를 제한해서 결과를 보여주려면 LIMIT 키워드를 사용하면 된다.







