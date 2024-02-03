
# @SpringBootTest

- webEnvironment 옵션 정리
	- 실제 컨테이너가 뜨는 옵션
	- mock 컨테이너가 뜨는 옵션 (뭐였는지 생각이 안나네..)

# @WebMvcTest
Controller 슬라이스 테스트를 위해 사용하는 테스트 어노테이션
- 서블릿 컨테이너를 mocking 한다.

~~~ java




~~~



https://we1cometomeanings.tistory.com/65
# @AutoConfigureMockMvc
서블릿 컨테이너를 mocking 하기 위해 사용하는 어노테이션

- @WebMvcTest와 @AutoConfigureMockMvc의 공통점은 테스트 시 실행되는 서블릿 컨테이너를 mocking 한다는 것이다. 
- @WebMvcTest 와 가장 큰 차이는 AutoConfigureMockMvc는 Controller 관련 bean들 외에도 Service와 Repository 관련 bean들도 생성해서 컨테이너에 등록해준다는 것이다.