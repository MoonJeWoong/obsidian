
![](https://i.imgur.com/mWg4sC3.jpeg)




# MVCC의 특징

![](https://i.imgur.com/9hLr3Wd.png)

- 트랜잭션 2에서 x에 쓰기 연산을 수행하기 위해 write lock을 걸고 자신의 스냅샷에 50이라는 값을 썼다고 가정해보자.
- 이후에 시작된 트랜잭션 1은 x 값을 읽을 때, DB에 커밋된 값만 읽을 수 있기 때문에 10이라는 값을 정상적으로 읽어오게 된다.


![](https://i.imgur.com/uAf5NYz.png)

- 트랜잭션 2가 commit된 이후에 트랜잭션 1이 x 값을 읽어온다면 이 때는 트랜잭션 1의 격리 수준에 따라서 다른 결과가 나온다.
- 만약 트랜잭션 1이 read commited 였다면 트랜잭션 2가 쓰기 작업 결과를 커밋한 이후이므로 변경된 결과값을 읽어오게 될 것이다.
- MySQL, PostgreSQL 모두 동일하게 동작한다.


![](https://i.imgur.com/8ZH71aH.png)

- 트랜잭션 1이 repeatable read 였다면  x 값을 읽어올 때 트랜잭션 시작 시간을 기점으로 읽어오기 때문에 50이 아니라 10을 읽어오게 된다.
- 예제에서는 편의상 repeatable read일 때 트랜잭션 시작 시간을 기준으로 커밋된 데이터를 읽는다고 설명되어 있지만 DB 마다 이 기준이 다르므로 공식 문서를 잘 확인해야 한다.
	- 트랜잭션 시작 시간 기준
	- 트랜잭션 내 읽기 혹은 쓰기 작업의 최초 발생 시간 기준


![](https://i.imgur.com/8Jk16Fp.png)
- repeatable read 는 MySQL, PostgreSQL 두 DBMS에서 모두 동일하게 동작한다.
- MVCC는 데이터를 읽을 때 트랜잭션 격리 수준에 따라 특정 시점을 기준을 정하고, 가장 최근에 commit 된 데이터를 읽는다.
- 이를 위해서 DBMS에서는 데이터 쓰기 이력을 별도로 관리해줘야 하기 때문에 추가적인 저장 공간을 필요로 한다.
- 이러한 MVCC를 바탕으로 DBMS에서는 서로 다른 트랜잭션에서 수행되는 읽기 작업과 스기 작업이 서로 간에 block을 하지 않아 전체적인 처리량을 향상시킬 수 있다는 장점을 취할 수 있다.

MySQL에서는 특정 시점을 기준으로 일관되게 데이터를 읽는 것을 Consistent Read 라고 한다.


![](https://i.imgur.com/10YBc6H.png)
![](https://i.imgur.com/9lnpZue.png)

- serializable 수준에서는 MySQL과 PostgreSQL의 동작 방식이 살짝 달라진다.
- MySQL에서는 serializable이 mvcc 보다는 lock을 기반으로 동작하게 되고, PostgreSQL에서는 SSI 기법이 적용된 mvcc로 동작하게 된다.
- 자세한 내용은 후술한다.


![](https://i.imgur.com/4YKIlNZ.png)

- read uncommited 수준은 mvcc가 일반적으로 적용되지 않는다.
- PostgreSQL에서는 read uncommited level이 존재하긴 하지만 read commited 처럼 동작한다.



# MVCC 환경에서 lost Update 해결하기


![](https://i.imgur.com/FRLSpt5.png)

- 트랜잭션 1이 먼저 시작되고 x 값을 읽고 쓰기 작업을 위해 락을 취득한 뒤 스냅샷에 쓰기 작업을 완료한다.
- 이후 트랜잭션 2가 시작되고 x 값을 읽은 뒤 쓰기 작업을 위해 락을 취득해야 하는데 이 때 트랜잭션 1이 락을 들고 있으므로 대기할 수 밖에 없게 된다.


![](https://i.imgur.com/IJ0jYcB.png)

- 이후 트랜잭션 1이 모든 작업을 마무리하고 결과 값들을 commit한다.
- x에 대한 쓰기 락을 대기하던 트랜잭션 2는 락을 취득하고 다음 작업을 진행할 수 있게 된다.

![](https://i.imgur.com/OEvkucZ.png)

- 결과 값을 놓고 보면 트랜잭션 1에서 수행한 업데이트가 씹혀서 데이터 정합성이 어긋나게 된 것을 알 수 있다. (lost update)


![](https://i.imgur.com/Ry4ks5f.png)

- 이 문제를 해결하기 위해 트랜잭션 2의 격리 수준을 repeatable read로 변경하자.


![](https://i.imgur.com/jhJ72u1.png)

- postgreSQL의 경우에는 repeatable read 수준에서 다른 트랜잭션이 같은 데이터에 먼저 update 작업을 커밋한 경우라면, 이후 트랜잭션을 롤백시킨다는 특징이 있다.


![](https://i.imgur.com/K23lsDB.png)

- 반대의 경우에도 lost update가 발생한다. 따라서 연관된 트랜잭션의 격리 수준도 같이 신경써줘야 함을 명심하자.


![](https://i.imgur.com/Rke5daI.png)

- 트랜잭션 1도 repeatable read로 격리 수준을 변경해서 lost update를 해결할 수 있다.


![](https://i.imgur.com/17GIsDs.png)

- 그런데 MySQL에서는 postgreSQL에서 처럼 repeatable read일 때 먼저 수행된 트랜잭션 내의 update를 고려해서 롤백시켜주는 기능이 없다.


![](https://i.imgur.com/Q73nrHd.png)
- 따라서 MySQL에서는 개발자가 직접 조회 쿼리를 쓸 때 for update를 붙여주는 Locking read를 사용해서 lost update 문제를 해결할 수 있다.


![](https://i.imgur.com/JmVdcfg.png)

- 트랜잭션 1번을 수행할 때도 Locking read를 이용해 읽기 작업을 수행한다. 그러면 트랜잭션 2번에서 x에 대해 Locking read를 하고 있기 때문에 해제될 때까지 대기하게 된다.


![](https://i.imgur.com/HotS9vM.png)

- 트랜잭션 2번의 작업 수행이 끝나고 x에 대한 읽기, 쓰기 락을 해제하면 대기하던 트랜잭션 1이 락을 취득하고 작업을 진행한다.
- x 값을 읽어올 때 Locking read 방식에서는 트랜잭션을 시작했을 때를 기점으로 커밋된 값을 읽어오는 것이 아니라 Locking read 시점에서 가장 최근 커밋된 값을 읽어오게 된다. 따라서 x 값이 기존의 값인 50이 아니라 트랜잭션 2에서 커밋한 80이라는 값이 읽어지게 되는 것이다.



![](https://i.imgur.com/B3axNO9.png)


- 이후 작업을 수행할 때에도 Locking read를 사용하면 lost update 문제 없이 정상적으로 작업을 완료할 수 있게 된다.
- MySQL에서 Locking read는 for update, for share 2가지 방식으로 사용될 수 있다.
	- 두 방식의 차이는 for update는 쓰기 락, for share는 읽기 락을 거는 것이다.



### Write Skew 현상 해결하기 (MySQL, postgreSQL)


![](https://i.imgur.com/iBNeehr.png)

![](https://i.imgur.com/n0CbaYt.png)

- 정상적인 결과가 나오지 않고 x와 y 모두 20으로 저장된 것을 확인할 수 있다.
- 이렇게 DBMS에 데이터 정합성이 깨진 상태로 값이 쓰여지는 현상을 write skew라고 한다.
- 이러한 현상은 MySQL, postgreSQL 둘 다 공통적으로 나타나게 된다.


![](https://i.imgur.com/2zcrb0E.png)

- MySQL에서 Write Skew 현상을 해결하기 위해서는 앞서 살펴본 Locking read를 사용한다.
- 그러면 트랜잭션 2번에서는 lock을 얻기 위해 트랜잭션 1번의 작업이 종료될 때 까지 대기하게 된다.

![](https://i.imgur.com/DYi6DXP.png)

- 트랜잭션 1번의 작업이 종료되고 x 값이 정상적으로 커밋되어 업데이트 된다.
- 이후 x의 락이 해제되고 트랜잭션 2번이 락을 취득하게 되어 작업이 진행된다.


![](https://i.imgur.com/pR11rY4.png)

- 트랜잭션 2번의 격리 수준은 repeatable read 이지만 Locking read는 격리 수준과 상관 없이 수행 시점 기준으로 가장 최근에 커밋된 값을 읽게 되므로 트랜잭션 1번에서 커밋한 값을 읽어온다.


![](https://i.imgur.com/yLF7Y9g.png)

- 결과적으로 트랜잭션 2번에서 Locking read를 사용해 x와 y 값을 읽어오게 되면 write skew 현상 없이 작업을 수행할 수 있다.




postgreSQL에서도 Locking read 기능을 사용해서 write skew 문제를 해결할 수 있는데 동작 방식이 조금 다르다.

![](https://i.imgur.com/Ns7iUo5.png)

- 앞서 살펴봤던 것처럼 postgreSQL에서는 repeatable read 수준일 때 앞선 트랜잭션에서 동일한 데이터에 대해 update를 커밋했다면 뒤 트랜잭션은 롤백을 처리한다.
- 결과적으로 트랜잭션 1번만 정상적으로 커밋되어 DBMS의 데이터 정합성을 유지하는 것이다.

![](https://i.imgur.com/FJGG2f8.png)

- 참고사항으로 read lock을 거는 for share를 사용하더라도 똑같이 롤백된다.




또 다른 해결방법으로는 트랜잭션 격리 수준을 serializable로 격상하는 것이다.

![](https://i.imgur.com/HO9nD1f.png)

- MySQL 에서 serializable 격리 수준은 repeatable read와 유사하나 평범한 조회 쿼리문에 대해 for share 방식으로 수행된다는 차이가 있다.
- 결과적으로 MVCC 방식 보다는 Lcok 기반의 방식으로 동작하게 된다고 일반적으로 얘기한다.
- 단점으로는 일반적인 쿼리문들이 for update 방식이 아니라 for share 방식으로 동작해서 데드락의 위험성이 더 높다는 것이 있어 이를 인지하고 사용하는 것이 좋다.


![](https://i.imgur.com/znZ4Pne.png)

PostgreSQL 방식에서는 SSI로 구현되어 MVCC 방식으로 동작한다고 하는데 자세한 내용은 직접 찾아봐야 할 것 같다.




---
Reference

[쉬운코드](https://www.youtube.com/watch?v=-kJ3fxqFmqA&list=PLcXyemr8ZeoREWGhhZi5FZs6cvymjIBVe&index=20)