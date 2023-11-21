
- Application Context

SpringBootTest를 수행할 때 Application을 로드해야 한다. 이 때 Application Context에 빈을 등록하기 위해서 @Bean으로 선언된 객체들을 찾는 등의 작업이 진행되기 떄문에 시간이 오래 걸린다. 


- @DirtiesContext 걷어내기

DirtiesContext 어노테이션 주석을 살펴보면 내용은 다음과 같다.
`테스트가 컨텍스트를 수정하는 경우 @DirtiesContext 어노테이션을 사용한다. 예를들어 싱글톤 빈의 상태를 변경한다던가, 내장된 DB 상태가 변경된다던가 하는 상황이 있을 수 있다. 뒤이어 요청된 테스트들은 새로운 컨텍스트를 제공받는다.`

테스트 간 데이터 격리라는 목적으로 @DirtiesContext를 사용하게 되면 모든 테스트마다 Context를 재생성하기 때문에 시간이 오래걸리게 되는 것이다. 이 대신 직접 데이터 격리를 수행할 수 있도록 DB를 수동으로 초기화시켜 줄 수 있다. 


``` java

public class DatabaseCleaner {

	private EntityManager entityManager;
	private List<String> tableNames;

	...

	@Transactional
	public void tableCleaner() {
		entityManager.flush();
		entityManager.clear();

		entityManager.createNativeQuery("SET REFERENTIAL_INTEGRITY FALSE").executeUpdate();  // 참조 무결성 검증 기능 OFF

		for (String tableName : tableNames) {
			entityManager.createNativeQuery("TRUNCATE TABLE " + tableName + " RESTART IDENTITY ").executeUpdate();
		}
		
		entityManager.createQuery("SET REFERENTIAL_INTEGRITY TRUE").executeUpdate();  // 참조 무결성 검증 기능 ON
	}
}

```



- Context Caching

일단 테스트 컨텍스트 프레임워크가 테스트를 위한 Application Context 혹은 WebApplicationContext를 load하면, 같은 테스트 셋 안에서 같은 고유 컨텍스트 설정을 요구하는 뒷 테스트들에 대해서 context가 캐싱되어 재사용된다. 캐싱이 어떻게 동작하는지 이해하기 위해서는 unique 와 test suite가 무슨 의미인지 이해하는 것이 중요하다.

Application Context는 로드될 때 사용된 설정 파라미터들의 조합으로 고유하게 식별될 수 있다. TestContext 프레임워크는 다음과 같은 설정 파라미터들을 사용해 컨텍스트 캐싱 키를 생성한다.

- locations (from @ContextConfiguration)
- classes (from @ContextConfiguration)
- contextInitializerClasses(from @ContextConfiguration)
- contextCustomizers(from ContextCustomizerFactory) - @DynamicPropertySource 메서드 뿐만 아니라 스프링 부트 테스트 지원에서 제공하는 @MockBean, @SpyBean등의 다양한 기능들도 포함된다.
- contextLoader(from @ContextConfiguration)
- parent(from @ContextHierarchy)
- activeProfiles(from @ActiveProfiles)
- propertySourceDescriptors(from @TestPropertySource)
- propertySourceProperties(from @TestPropertySource)
- resourceBasePath (from @WebAppConfiguration)

예를 들어 TestClassA에서 @ContextConfiguration 어노테이션의 옵션으로 locations 혹은 value에 {"app-config.xml", "test-config.xml"}라고 명시를 했다면, ApplicationConext를 로드하고 locations 옵션 하나를 기반으로 한 key에 매핑해서 static context cache에 저장한다. 이후 TestClassB에서 locations 외 다른 옵션들을 설정해주지 않으면 동일한 ApplicationContext를 사용하는 것으로 간주하고 캐싱된 context를 사용한다.
