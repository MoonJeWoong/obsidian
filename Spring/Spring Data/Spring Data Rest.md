
Spring Data JPA를 통해 사용할 수 있는 JpaRepository 인터페이스를 상속하는 Repository 인터페이스를 만들면 이를 기반으로 CRUD Rest API를 자동으로 빠르게 생성할 수 있게 도와주는 Spring Data의 프로젝트이다.

Spring data를 사용하기 위해서는 다음과 같은 gradle 의존성 추가와 yaml 설정이 필요하다.

~~~ gradle
implementation 'org.springframework.boot:spring-boot-starter-data-rest'  

// Spring Data Rest 로 제공되는 API들을 웹에서 편하게 접근하고 테스트할 수 있도록 도와주는 외부 라이브러리
implementation 'org.springframework.data:spring-data-rest-hal-explorer'
~~~

~~~ yaml
Spring:
  data:  
    rest:  
      base-path: /api  
      detection-strategy: annotated   # @RepositoryRestResource 어노테이션을 선언한 Repository 인터페이스만 API로 노출한다.

~~~


~~~ java
// 도메인 클래스 
@Entity 
public class Person { 
	@Id @GeneratedValue(strategy = GenerationType.AUTO) 
	private Long id; 
	private String firstName; 
	private String lastName; // getters and setters 
} 

// 레포지토리 인터페이스 
@RepositoryRestResource
public interface PersonRepository extends JpaRepository<Person, Long> { 
}
~~~

기본적인 data의 서빙은 Spring Data Rest를 통해 수행하고 복잡한 데이터를 조합해서 제공해야 하는 경우에는 따로 API를 추가하는 방식으로도 개발할 수 있다.



### Spring Data Rest 정상 동작 테스트 시 신경쓸 것

~~~ java

@SuppressWarnings("NonAsciiCharacters")  
@DisplayNameGeneration(ReplaceUnderscores.class)  
@WebMvcTest  
public class DataRestTest {  
  
    @Autowired  
    private MockMvc mockMvc;  
  
    @Test  
    void 정상_동작_테스트() throws Exception {  
        // when & then  
        mockMvc.perform(get("/api/articles"))  
                .andExpect(status().isOk())  
                .andExpect(content().contentType(MediaType.valueOf("application/hal+json")));  
    }  
}
~~~

위 테스트는 정상 동작하지 않는다. Spring Data Rest가 정상적으로 동작하기 위해서는 Repository 인터페이스 기반으로 빈이 생성되어 컨테이너에 등록되어야 하는데 @WebMvcTest에서는 Controller와 관련된 bean만 로드하기 때문에 테스트가 실패하게 되는 것이다. 즉 Spring Data Rest 가 수행해야 하는 auto configuration을 진행하지 않은 것이다.

=> 해결책은 간단하게 @SpringBootTest + @AutoConfigureMockMvc 조합을 사용하는 것이다.



