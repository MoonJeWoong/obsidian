
DB를 사용하는 4가지 특징
구구가 얘기해준 DB 각 속성들을 제공하기 위한 실제 기술들을 예시로 학습해두자. 

- Atomicity : 원자성
	- 일련의 여러 작업들이 하나의 단위로 묶여서 처리되도록 하는 속성으로 모두 성공하거나 모두 실패하는 방식으로 원자성을 보장한다.
- Consistency : 일관성
	- 트랜잭션은 DB 상태를 consistent 상태에서 또 다른 consistent 상태로 바꿔줘야 한다.
		- consistent 상태란? <mark style="background: #FFF3A3A6;">무결성 제약 조건등을 반하지 않는 DB의 상태</mark>
	- constraints, trigger 등을 통해 DB에 정의된 rule들을 트랜잭션이 위반하면 안된다. 
	- 트랜잭션이 DB에 정의된 rule들을 위반했는지는 DBMS가 커밋하기 전에 확인한다.
	- 이 외에도 개발자는 트랜잭션이 consistent하게 동작하는지 확인해야 한다.
- Isolation : 독립성
	- 트랜잭션의 격리 수준을 다르게 설정할 수 있게 함으로서 동시 처리를 통한 성능을 보장하면서도 독립적인 트랜잭션으로 수행하는 것처럼 하여 동시 처리로 인한 문제를 방지한다. 
	- DBMS는 이를 위해 여러 종류의 Isolation level을 제공한다. 
	- Concurrency Control은 이러한 Isolation level을 보장하기 위한 것이다. 
- Durability : 회복성
	- 한 번 commit 되어 DB에 저장된 데이터는 반드시 하드웨어에 물리적으로(영구적으로) 기록되어야 한다. 만약 DB 전원 장치가 갑자기 나간다고 하더라도 커밋한 작업 내용은 완전하게 저장되어 다시 복구할 수 있어야 한다.
	- 영구적으로 저장한다고 할때는 일반적으로 비휘발성 메모리 (HDD, SSD 등...)에 저장함을 의미한다.
	- 기본적으로 트랜잭션의 durability는 DBMS가 보장한다.