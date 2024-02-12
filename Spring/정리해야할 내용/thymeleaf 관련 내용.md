
### 스프링 부트 thymeleaf 기본 설정

~~~ java
spring:  
	thymeleaf:  
		prefix: classpath:/templates/  
		suffix: .html
~~~

스프링 부트 thymeleaf에서는 기본 설정되어 있는 prefix와 suffix를 통해 렌더링할 view를 찾는다.
- resources:templates/ +{ViewName} + .html 

참고 : https://docs.spring.io/spring-boot/docs/2.1.7.RELEASE/reference/html/common-applicationproperties.html




