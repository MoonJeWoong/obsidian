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


# JOIN

## 조건에 맞는 도서와 저자 리스트 출력하기

``` SQL
SELECT BOOK_ID, AUTHOR_NAME, DATE_FORMAT(PUBLISHED_DATE, '%Y-%m-%d') AS PUBLISHED_DATE
FROM BOOK INNER JOIN AUTHOR ON BOOK.AUTHOR_ID = AUTHOR.AUTHOR_ID
WHERE BOOK.CATEGORY = '경제'
ORDER BY PUBLISHED_DATE;
```

- 날짜 형식을 지정해야 할 때는 DATE_FORMAT 메서드를 사용한다.
- A INNER JOIN B ON A.id = B.id 와 같은 형식으로 조인 시 매핑할 컬럼을 지정할 수 있다.
	- ON 절 사용하는 방법을 잘 숙지하자.