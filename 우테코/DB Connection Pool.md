
# 0단계 학습테스트 - DataSource 다루기

- DriverManager
JDBC 드라이버를 관리하는 가장 기본적인 방법으로 커넥션 풀, 분산 트랜잭션 등을 지원하지 않아서 잘 사용하지 않습니다.

JDBC 4.0 이전에는 적절한 JDBC 드라이버를 직접 이름으로 찾아서 적용시켜줘야 하는 불편함이 있었습니다.
JDBC 4.0 이후부터는 DriverManager가 적절한 JDBC 드라이버를 찾는 방식으로 기능이 개선되었습니다.

``` java
DriverManager.getConnection(H2_URL, USER, PASSWORD);
```

- DataSource
DataSource는 데이터베이스, 파일 등과 같은 물리적 데이터 소스에 연결할 때 사용하는 인터페이스입니다. 
구현체는 각 vendor사마다 별도로 제공합니다.
JdbcDataSource 클래스는 H2 데이터베이스에서 제공하는 구현체 클래스입니다.

DriverManager가 아니라 DataSource를 사용하는 이유
- 애플리케이션 코드를 수정하는 것이 아니라 properties 설정을 통해 DB 연결 설정을 변경할 수 있기 때문이다.
- DB 커넥션 풀, 분산 트랜잭션 등의 기능을 제공하기 때문이다.


``` java
final JdbcDataSource dataSource = new JdbcDataSource();
datasource.setURL("jdbc:h2:./test");
datasource.setUser("sa");
dataSource.setPassword("");

try (final var connection = dataSource.getConnection()) {
	// connection task
}
```



# 1단계 커넥션 풀링

- 커넥션 풀링
커넥션을 생성하는 것은 외부와 통신하기 위한 I/O 작업을 해주는 리소스 비용이 발생하게 되는데 이 비용이 많이 비쌉니다.
따라서 커넥션을 매번 생성하는 것이 아니라 미리 커넥션을 만들어두고 그 커넥션들을 계속 재활용하는 것이 효율적인데 이를 수행해주는 것이 커넥션 풀입니다.

## Connection Pooling and Statement Pooling
https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/javax/sql/package-summary.html

미들 티어 커넥션 풀 매니저와 함께 동작하는 DataSource를 통해 생성된 커넥션들은 커넥션 풀에 등록됩니다. 새로 커넥션을 생성하는 것은 비싼 비용이 드는 작업인데 한 번 생성된 커넥션을 커넥션 풀을 통해 재활용함으로써 새로운 커넥션을 생성하는 횟수를 크게 줄일 수 있습니다.

커넥션 풀은 사용자 단으로부터 완벽하게 은닉화됩니다. 미들 티어의 Java EE 설정에서 자동으로 수행되며 그렇기 때문에 애플리케이션 단에서는 코드적인 변화가 없습니다. 애플리케이션은 DataSource.get Connection 메서드를 통해 풀에 등록된 커넥션을 얻고 사용하기만 하면 됩니다.

커넥션 풀을 사용하는 클래스와 인터페이스들
- `ConnectionPoolDataSource`
- `PooledConnection`
- `ConnectionEvent`
- `ConnectionEventListener`
- `StatementEvent`
- `StatementEventListener`

커넥션 풀 매니저는 뒷단에서 이러한 클래스와 인터페이스들을 사용합니다. ConnectionPoolDataSource 객체가 PooledConnection 객체를 생성할 때 커넥션 풀 매니저는 ConnectionEventListener 객체로서 새로운 PooledConnection 객체를 등록하게 됩니다. 이후 커넥션이 종료되거나 에러가 발생하는 경우, 커넥션 풀 매니저는 ConnectionEvent 객체를 포함하는 알림을 받게 됩니다.

만약 커넥션 풀 매니저가 PreparedStatements 객체에 대한 Statement 풀링을 지원한다면, 위에서 동작한 방식과 비슷하게 StatementEventListener 객체로서 새로운 PooledConnection 객체를 등록하며 PreparedStatement가 종료되거나 에러가 발생할 때 StatementEvent 객체와 함께 알림을 받게 됩니다. 


# 스프링 부트에서 지원하는 커넥션 풀
- HikariCP
- Tomcat pooling Datasource
- Commons DBCP2
- Oracle UCP
위와 같은 순서대로 우선순위를 가지고 선택해서 사용하게 된다.

- spring.datasource.type 프로퍼티들을 설정한 값으로 커넥션 풀을 사용할 수 있습니다.
- 추가적인 connection pool들도 DatasourceBuilder를 통해 수동으로 설정될 수 있습니다. 만약 커스텀된 DataSource Bean을 정의한다면 auto-configuration은 발생하지 않습니다. 다음과 같은 커넥션 풀들이 DataSourceBuilder를 통해 지원됩니다.

- HikariCP
- Tomcat pooling `Datasource`
- Commons DBCP2
- Oracle UCP & `OracleDataSource`
- Spring Framework’s `SimpleDriverDataSource`
- H2 `JdbcDataSource`
- PostgreSQL `PGSimpleDataSource`
- C3P0

스프링 부트 2.0부터 Hikari CP를 기본으로 사용하는 이유?
- 성능과 동시성이 다른 CP에 비해 매우 뛰어나다.


# HikariCP 설정하기
https://github.com/brettwooldridge/HikariCP#rocket-initialization

``` java
final var hikariConfig = new HikariConfig();
hikariConfig.setJdbcUrl(H2_URL);
hikariConfig.setUsername(USER);
hikariConfig.setPassword(PASSWORD);
hikariConfig.addDataSourceProperty("cachePrepStmts", "true")
hikariConfig.addDataSourceProperty("prepStmtCacheSize", "250")
hikariConfig.addDataSourceProperty("prepStmtCacheSqlLimit", "2048")

final var dataSource = new HikariDataSource(hikariConfig);
final var properties = dataSource.getDataSourceProperties();
```



### 커넥션 풀 사이징에 대하여
https://github.com/brettwooldridge/HikariCP/wiki/About-Pool-Sizing
- connections = ((core_count * 2) + effective_spindle_count)

core_count = cpu 코어 개수
effective_spindle_count = 동시 처리 가능한 하드디스크 개수


실험 블로그
https://hyuntaeknote.tistory.com/12


### MySQL Configuration

MySQL로부터 최적의 성능을 내기 위해 추천되는 세팅들이 공식 위키에 등재되어 있습니다.

- prepStmtCacheSize
해당 옵션은 MySQL 드라이버가 커넥션 당 캐싱할 prepared statement 개수입니다. 기본 값은 보수적인 25이며 추천 세팅 값은 250 - 500 사이입니다.


- prepStmtCacheSqlLimit
해당 옵션은 드라이버가 cache하게 될 prepared SQL statement의 최대 길이입니다. MySQL 기본 값은 256인데 Hibernate와 같은 ORM 프레임워크를 사용하는 경우 특히 더 이 길이는 부족한 경향을 보입니다. 권장 세팅값은 2048입니다.


- cachePrepStmts
위에서 언급한 파라미터들은 cache 설정 값이 기본적으로 false이기 때문에 이를 true로 세팅해주지 않으면 아무런 의미가 없게 됨에 유의해야 합니다.

- useServerPrepStmts
MySQL에서 제공하는 server-side prepared statement 지원으로 상당한 성능 부스트를 제공하므로 true로 설정값을 세팅하세요.


# 2단계 HikariCP 설정하기

SpringBoot 에서 설정 파일인 application.yml을 사용하여 DataSource를 설정할 수 있다.
하지만 DataSource를 여러 개 사용하거나 세부설정을 하려면 빈을 직접 생성하는 방법을 사용한다.

