

# datasource 관련 설정

~~~ yaml

spring:  
  # datasource 관련 설정  
  datasource:  
#    url: jdbc:mysql://localhost:3306/board  
#    username: woong  
#    password: woong  
#    driver-class-name: com.mysql.cj.jdbc.Driver  

    url: jdbc:h2:mem:testdb  
    username: sa  
    driver-class-name: org.h2.Driver

h2:
  console:
    enabled: true

~~~

- datasource에서 주석 처리된 부분으로 하면 mysql 도커 컨테이너로 실제 커넥션을 맺어 서버를 띄울 수 있지만 후자는 인메모리 h2 DB로 띄울 수 있다.
- h2.console.enabled 옵션을 true로 주면 스프링 부트 서버를 띄우고 나서 h2 console 접근이 가능하다. 드라이버만 적절하게 변경해주면 mysql DB에도 콘솔 내에서 접근이 가능하다.