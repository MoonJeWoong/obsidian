
https://www.baeldung.com/integration-testing-in-spring

Spring MVC를 이용해서 서블릿 컨테이너를 실제로 실행하지 않고 컨트롤러를 테스트하는 방법에 대한 튜토리얼 정리


# Spring MVC Test Configuration


### Junit5를 사용한 테스트에서 스프링 적용하기

Junit5는 한 클래스에서 JunitTest가 수행될 수 있도록 도와주는 확장 인터페이스를 정의한다.

JunitTest를 사용하기 위한 클래스 확장을 하기 위해서는 다음과 같은 사항들이 수행되어야 한다.
- 테스트 클래스에 @ExtendWith 어노테이션을 선언
- 로드할 확장 클래스를 명시

~~~ java

@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes = { ApplicationConfig.class })
@WebAppConfiguration
public class GreetControllerIntegrationTest {
    ....
}

~~~

위 예시에서 @ContextConfiguration 어노테이션은 테스트하는데 필요한 context를 configuration하고 load 시켜주는 역할을 수행한다. (필수는 아님, 테스트에 필요한 configuration을 진행하기 위해 추가한 듯 하다.)

@WebAppConfiguration : 웹 어플리케이션 context를 로드하기 위한 어노테이션

