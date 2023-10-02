
Mockito는 모의 객체의 생성, 검증, 스텁을 지원하는 자바 모의 객체 프레임워크이다.

# gradle 의존성 추가하기

``` gradle
dependencies {  
    testImplementation "org.assertj:assertj-core:3.24.2"  
    testImplementation "org.junit.jupiter:junit-jupiter-api:5.7.2"  
    testImplementation "org.mockito:mockito-core:5.4.0"  
    testRuntimeOnly "org.junit.jupiter:junit-jupiter-engine:5.7.2"  
}
```

# 모의객체 생성

``` java
import static org.mockito.BDDMockito.given;
import static org.mockito.Mockito.mock;

class JdbcTemplateTest {  
  
    private JdbcTemplate jdbcTemplate;  
  
    @BeforeEach  
    void before() throws SQLException {  
        Connection connectionMock = mock(Connection.class);  
        DataSource dataSourceMock = mock(DataSource.class); 
        // BDDMockito.given()을 사용한 스텁 설정
        given(dataSourceMock.getConnection("", "")).willReturn(connectionMock);  
		// 특정 타입의 익셉션을 발생시키도록 스텁 설정
		given(dataSourceMock.getConnection("exceptionUser", "ExceptionPwd"))  
        .willThrow(IllegalArgumentException.class);

        jdbcTemplate = new JdbcTemplate(dataSourceMock);  
    }  
  
    @Test  
    void updateTest() {  
  
    }  
}

```


``` java
// 반환 타입이 void인 경우 exception 발생 모킹 처리 방식
List<String> listMock = mock(List.class);  
willThrow(UnsupportedOperationException.class)  
        .given(listMock)  
        .clear();
```


- given().willREturn()을 원하는 만큼 체이닝해서 적용하면 원하는 횟수만큼 값을 반환하도록 할 수 있다.
``` java
given(resultSetMock.next())
    .willReturn(true)
    .willReturn(true)
    .willReturn(false);
given(rowMapperMock.mapRow(eq(resultSetMock), anyInt()))
    .willReturn(new TestUser(1L, "hiiro", "hiiro", "hiiro"))
    .willReturn(new TestUser(2L, "ako", "ako", "ako"));
```

---


# 인자매칭 처리

``` java
import static org.mockito.BDDMockito.given;
import static org.mockito.Mockito.mock;

given(genMock.generate(GameLevel.EASY)).willReturn("123");
String num = genMock.generate(GameLevel.NORMAL);
```

여기에서 메서드 실행 결과 값인 num은 null이 반환된다.
Mockito는 기본적으로 스텁 설정이 안되어 있으면 리턴 타입의 기본 값을 반환한다. 예를 들어서 리턴 타입이 primitive 타입인 경우 각 타입의 기본 값을, reference 타입이라면 null을 반환한다.


### ArgumentMatchers

ArgumentMatchers 클래스를 사용하면 정확하게 일치하는 값 대신 임의의 값을 받을 수 있도록 스터빙할 수 있다.

``` java
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.BDDMockito.given;
import static org.mockito.Mockito.mock;

@Test  
void queryForObjectTest() throws SQLException {  
    //given  
    String account = "account";  
    String password = "pwd";  
    String email = "email";  
    String sql = "select id, account, password, email from users where id = ?";  
  
    ResultSet resultSetMock = mock(ResultSet.class);  
    RowMapper<TestUser> rowMapperMock = mock(RowMapper.class);  
    given(preparedStatementMock.executeQuery()).willReturn(resultSetMock);  
    given(resultSetMock.next()).willReturn(true);  
    given(rowMapperMock.mapRow(eq(resultSetMock), anyInt()))  
            .willReturn(new TestUser(1L, account, password, email));  
  
    //when  
    TestUser testUser = jdbcTemplate.queryForObject(sql, rowMapperMock, 1L);  
  
    //then  
    SoftAssertions.assertSoftly(softly -> {  
        softly.assertThat(testUser.getAccount()).isEqualTo(account);  
        softly.assertThat(testUser.getPassword()).isEqualTo(password);  
        softly.assertThat(testUser.getEmail()).isEqualTo(email);  
    });  
}

```

ArgumentMatchers 클래스에서 제공하는 인자 매칭 처리 메서드는 다음과 같다.

- 기본 데이터 타입에 대한 임의 값 일치 : anyInt(), anyShort(), ... anyBoolean()
- 문자열 값에 대한 임의 값 일치 : anyString()
- 임의 타입에 대한 일치 : any()
- 임의 컬렉션에 대한 일치 : anyList(), anyMap(), anySet(), anyCollection()
- 정규표현식을 이용한 String 값 일치 여부 : matches(String), matches(Pattern)
- 특정 값과의 정확한 일치 여부 : eq(값)

주의해야 할 점으로는 스터빙을 진행할 때 ArgumentMatchers 클래스의 인자 매칭 메서드를 하나라도 사용하면 해당 스터빙에서는 모두 ArgumentMatcher를 이용해서 인자 매칭해줘야 한다. 따라서 임의의 값과 정확하게 일치하는 인자를 사용하고 싶다면 eq() 메서드를 통해 특정 값을 정확하게 지정해주면 된다.


# 행위 검증

모의 객체의 역할 중 하나는 실제로 모의 객체가 호출되었는지 검증하는 것이다. 

``` java
import static org.mockito.BDDMockito.then;
import static org.mockito.Mockito.mock;

public class GameTest {
	
	@Test
	void init() {
		GameNumGen genMock = mock(GameNumGen.class);
		Game game = new Game(genMock);
		game.init(GameLevel.EASY);

		then(genMock).should().generate(GameLevel.EASY);
	}
}
```

위 예제는 genMock 객체의 generate 메서드가 GameLevel.EASY 인자를 받아서 수행되었는지를 검증한다. 정확한 값이 아니라 호출 여부 자체가 검증 사항이라면 다음과 같이 any()를 활용하자.

``` java
then(genMock).should().generate(any());
```

모의 객체의 호출 횟수를 검증하고 싶을 때는 다음과 같은 메서드들을 활용할 수 있다.

``` java
// generate 메서드를 한 번만 호출하는지 검증
then(genMock).should(only()).generate(any())  
```

- only(): 한 번만 호출
- times(int): 지정한 횟수만큼 호출
- never(): 호출하지 않음
- atLeast(int): 적어도 지정한 횟수만큼 호출
- atLeastOnce(): atLeast(1)과 동일
- atMost(int): 최대 지정한 횟수만큼 호출


# 인자 캡처

단위테스트를 진행하다보면 모의 객체를 호출할 때 사용한 인자 자체를 검증해야 할 때가 있다. Mockito의 ArgumentCaptor를 사용하면 메서드 호출 여부를 검증하는 과정에서 실제 호출할 때 전달한 인자를 보관할 수 있다.

``` java

@Test  
void testEmail() {  
    //given  
    Member memberMock = mock(Member.class);  
    memberMock.addNaverEmail("munjin0201");  
    ArgumentCaptor<String> captor = ArgumentCaptor.forClass(String.class);  
  
    //when  
    then(memberMock).should().addNaverEmail(captor.capture());  
  
    //then  
    assertThat(captor.getValue()).isEqualTo("munjin0201");  
}
```