
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