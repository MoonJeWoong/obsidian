
Lock을 사용해서 동시성 제어를 어떻게 구현할 수 있는가?

![[Pasted image 20231109152200.png]]

- DB에 저장된 x라는 변수에 10이라는 값이 저장되어 있는 상황에서 트랜잭션 1과 트랜잭션 2가 각각 다른 값을 x에 write 하려는 상황이다.
- write 작업은 단순하게 값 하나를 바꾸는 것보다 더 복잡한 과정이다. 
	- 해당 데이터에 index가 걸려있다면 관련 데이터 처리도 함께 이뤄져야 한다.
	- 값이 변경된 이후 파일 등에 함께 정보가 갱신되어야 할 수도 있다.
- 이런 상황에서 같은 데이터에 읽기/쓰기 작업이 동시에 발생한다면 예상한 의도와는 다르게 동작할 가능성이 높다.
- 읽기 / 쓰기 작업의 동시성 문제를 해결하기 위해서 Lock 개념을 도입할 수 있다.
	- 해당 작업을 진행하기 위해서는 읽기 / 쓰기 Lcok을 취득해야만 한다.

두 트랜잭션이 동시에 쓰기를 진행하려 하는 경우 Lock 동작 방식

- wirte lock을 먼저 획득한 트랜잭션의 작업이 진행되는 동안 write_lock(x)를 실행한 다른 트랜잭션은 block된다.
- 이후 write lock을 선점한 트랜잭션의 작업이 완료되면 block되어 있던 트랜잭션이 write lock을 얻게 되어 작업을 수행할 수 있게 된다.


### write_lock(x)

- write lock 혹은 Exclusive Lock 이라고 한다.
- read / write 작업을 수행하기 전에 사용한다.
- 다른 트랜잭션에서 데이터를 read/write 하는 것을 허용하지 않는다.
- write_lock을 트랜잭션의 쓰기 작업 뿐만 아니라 읽기 작업을 진행할 때에도 사용할 수 있음을 주의하자! (즉 읽기 작업을 진행하기 위해 write_lock을 획득해서 진행할 수 있는 것이다.)


### read_lock(x)

- read lock 혹은 Shared Lock 이라고 한다.
- read 작업을 수행할 때 사용한다.
- 다른 트랜잭션에서 같은 데이터를 read 하는 것을 허용한다.
- write는 허용하지 않는다.



# Lock 호환성

|            | read-lock | write-lock |
|:---------- |:--------- |:---------- |
| read-lock  | O         | X          |
| write-lock | X         | X          |



Lock을 사용해서 어떻게 트랜잭션의 isolation 레벨을 보장할 수 있을까?
Lock을 사용해서 어떻게 동시성을 제어하고 serializability를 보장할 수 있을까?


# Lock만 사용한다고 해서 Serializability가 보장되는 것은 아니다.


DB : x=100, y=200 인 상태라고 가정해보자.

| 트랜잭션 1 (x와 y의 합을 x에 저장) | 트랜잭션 2 (x와 y의 합을 y에 저장) |
|:---------------------------------- |:---------------------------------- |
| read_lock(y)                       | read_lock(x)                       |
| read(y)                            | read(x)                            |
| unlock(y)                          | unlock(x)                          |
| write_lock(x)                      | write_lock(y)                      |
| read(x)                            | read(y)                            |
| write(x = x + y)                   | write(y = x + y)                   |
| unlock(x)                          | unlock(y)                          |


![[Pasted image 20231109165922.png]]


![[Pasted image 20231109170027.png]]

1번 트랜잭션이 수행된 후 2번 트랜잭션이 수행되는 경우와 그 반대의 경우 결과 값은 달라진다.


![[Pasted image 20231109170132.png]]

위의 상황과 다르게 각 트랜잭션이 동시에 수행되는 경우 살펴보면 2번 트랜잭션이 먼저 수행되는 경우의 결과 값과 완전히 다른 결과가 도출되어 Lock을 사용해도 serializability가 보장되고 있지 않음을 확인할 수 있다. (Nonserializable 하다)

이렇게 되는 원인은 2번 트랜잭션에서 y 값을 업데이트 하기 위해서 x 값을 읽었는데 1번 트랜잭션에서 아직 업데이트 되기 이전의 y 값을 읽어버리기 때문에 Nonserializable 해지는 것이다.

![[Pasted image 20231109171211.png]]


이를 해결하기 위해 2번 트랜잭션에서 x 에 대한 read lock을 unlock 하기 전에 y 에 대한 write lock을 걸어주면 Serializable하게 작업을 수행할 수 있다.

![[Pasted image 20231109171309.png]]


정리하자면 아래 이미지와 같이 기존 트랜잭션들의 작업에서 lock을 획득하고 해제하는 순서를 변경해줘서 작업들이 serializable 하게 수행될 수 있다.

![[Pasted image 20231109171845.png]]



# 2 Phase Locking Protocol

이제 Lock과 관련된 작업들만 뽑아 놓고 한 번 살펴보자

![[Pasted image 20231109172020.png]]

정리하자면 한 트랜잭션에서 필요로 하는 모든 Locking Operation이 최초의 Unlock Operation 보다 먼저 수행되도록 하는 방법이 2 phase locking protocol이다. 즉 한 번 unlock을 시작하면 새로운 lock을 획득하지 않는 것이라고도 말할 수 있다.

2 phase locking protocol을 사용하면 Serializability를 보장할 수 있다.


### 2 Phase Locking Protocol 에서 발생할 수 있는 DeadLock

![[Pasted image 20231109172611.png]]

각 트랜잭션이 Expanding Phase에서 필요한 lock들을 얻어 가는 과정이 위 그림과 같이 동시에 발생한다고 가정해보자.

트랜잭션 1에서는 y에 대한 read lock을 획득하고 x에 대한 write lock을 획득하려고 한다. 그런데 만약 트랜잭션 2에서 x에 대한 read lock을 먼저 획득한다면, 트랜잭션 1은 x에 대한 write lock을 얻기 위해 block당하게 된다. 그리고 트랜잭션 2 또한 Expanding Phase에서 필요한 y에 대한 write lock을 획득하고자 하나 트랜잭션 1에서 이미 획득한 상태이기에 block 당하게 된다. 결과적으로 두 트랜잭션 모두 Expanding Phase를 더 이상 진행하지 못하고 계속 block 상태로 유지되는 DeadLock이 발생하게 된다.




# 2 Phase Locking Protocol 종류


### Conservative 2PL

모든 lock을 취득한 뒤 트랜잭션 작업을 진행하는 방식
- 데드락의 위험으로부터 자유롭다
- 하지만 트랜잭션이 필요로 하는 모든 lock을 취득한 이후에야 트랜잭션 수행이 가능하기 때문에  트랜잭션 시작 자체가 어려워져서 실용적인 방법은 아니다.


### Strict 2PL

- strict schedule을 보장하는 2PL
	- strict schedule 이란 rollback이 발생할 때 이상 현상이 발생하지 않도록 하기 위해 권장되는 스케쥴링 방식 중 가장 엄격한 방식이다.
	- lock을 취득한 트랜잭션의 작업이 완전히 commit되거나 rollback 되기 전까지 다른 트랜잭션에서 해당 데이터에 대해 읽기나 쓰기 작업을 일체 진행하지 않는 것이다.
- recoverability를 보장
- write-lock을 commit / rollback 할 때 반환한다.


### String Strict 2PL

- strict schedule을 보장하는 2PL
- recoverability를 보장
- write-lock 뿐만 아니라 read-lock 또한 트랜잭션이 commit / rollback 완료된 이후 반환하는 방식이다.
- Strict 2PL 방식보다 구현하기 쉽다는 장점이 있다.
- read lock을 갖고 있는 시간 또한 길어지기 때문에 다른 트랜잭션이 더 오래 대기해야 할 수 있다는 단점이 있다.


# 2PL Protocol의 문제점

두 트랜잭션이 동시에 수행될 때 양 쪽 모두 read 가 아닌 이상 다른 한 쪽의 트랜잭션은 block 될 수 밖에 없기 때문에 전체 처리량이 떨어질 수 밖에 없다. 

두 개 이상의 트랜잭션들이 동시에 write 하는 것은 어쩔 수 없지만 read - write 트랜잭션이 block 되는 문제는 해결해보자는 배경에서 등장한 개념이 MVCC 이다.


# MVCC (Multiversion Concurrency Control)

오늘날 대부분의 관계형 데이터베이스에서 Lock과 MVCC를 혼용해서 사용한다.