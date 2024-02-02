
https://www.baeldung.com/spring-boot-data-sql-and-schema-sql


~~~ yaml
defer-datasource-initialization: true
~~~

- 일반적으로 data.sql 파일이 resources 하위에 존재하면 스프링이 이를 감지하고 DB 초기화를 진행해준다. 이 때 초기화 시점이 하이버네이트 초기화 시점보다 앞선다. 일반적으로 하이버네이트에서 엔티티에 대응되는 테이블을 생성하는 초기화 과정을 거친 뒤에 데이터가 삽입되어야 하므로 defer-datasource-initialization 옵션을 true로 줘서 하이버네이트 초기화 완료 이후 data.sql 을 통한 데이터 삽입이 이뤄지도록 해야 한다.


~~~ yaml
spring.sql.init.mode: always
~~~

- data.sql 혹은 schema.sql 을 이용한 스크립트 기반 db 초기화를 원하는 경우 위 옵션을 활성화 해줘야 한다. 임베디드 DB를 사용하는 경우 이 옵션의 기본 값이 always로 설정된다.