
testEntityManager는 내부적으로 entityManagerFactory를 포함하고 있어서 동작 시에 entityManagerFactory로부터 entityManager를 반환받아 작업을 처리하게 된다.

testEntityManager가 entityManager와 다른 점은 크게 2가지로 정리할 수 있다.

- 테스트 시 유용하게 사용할 수 있는 추가적인 메서드를 몇 가지 더 제공한다. 
	- persist, flush, clear 등의 동작은 결과적으로 entityManager를 factory로부터 반환받아서 수행하게 된다.
- @SpringBootTest 시에는 entityManager 타입으로 빈이 등록되지만, @DataJpaTest 시에는 TestEntityManager 타입으로 빈을 등록해서 주입받아 테스트에 사용할 수 있다.


