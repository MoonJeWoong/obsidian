
JPA 자체에서 제공하는 Auditing을 사용할 수 있고, Data JPA에서 제공하는 Auditing을 사용할 수도 있다. @Auditing 어노테이션 자체는 Spring Data JPA의 기능이다.

- JPA 자체 Auditing 사용하기
	- BaseEntity 설계
    
    ```java
    @MappedSuperclass
    @Getter
    public class JpaBaseEntity {
    
        @Column(updatable = false)
        private LocalDateTime createdDate;
        private LocalDateTime updatedDate;
    
        @PrePersist
        public void prePersist() {
            LocalDateTime now = LocalDateTime.now();
            createdDate = now;
            updatedDate = now;
        }
    
        @PreUpdate
        public void preUpdate() {
            updatedDate = LocalDateTime.now();
        }
    }
    ```
    
    @PrePersist, @PostPersist, @PreUpdate, @PostUpdate ⇒ JPA 주요 이벤트 어노테이션!

---

- Spring Data JPA Auditing 사용하기
    
    - 메인 Application파일에 @EnableJpaAuditing 선언하기
    ```java
    @EnableJpaAuditing
    @SpringBootApplication
    public class DataJpaApplication {
    
    	public static void main(String[] args) {
    		SpringApplication.run(DataJpaApplication.class, args);
    	}
    }
    ```


    - BaseEntity에 @EntityListeners(AuditingEntityListener.class) 선언하기
	    - AuditingEntityListener는 Spring Data JPA에서 제공하는 엔티티의 영속, 수정을 감지하는 이벤트 리스너이다.
    
    ```java
    @EntityListeners(AuditingEntityListener.class)
    @MappedSuperclass
    @Getter
    public class BaseTimeEntity {
    
        @CreatedDate
        @Column(updatable = false)
        private LocalDateTime createdDate;
    
        @LastModifiedDate
        private LocalDateTime lastModifiedDate;
    }
    
    @EntityListeners(AuditingEntityListener.class)
    @MappedSuperclass
    @Getter
    public class BaseEntity extends BaseTimeEntity {
    
        @CreatedBy
        @Column(updatable = false)
        private String createdBy;
    
        @LastModifiedBy
        private String modifiedBy;
    }
    ```
    
    - 등록자, 수정자 처리해주는 AuditorAware 스프링 빈 등록하기
    
    ```java
    @EnableJpaAuditing
    @SpringBootApplication
    public class DataJpaApplication {
    
    	public static void main(String[] args) {
    		SpringApplication.run(DataJpaApplication.class, args);
    	}
    
    	@Bean
    	public AuditorAware<String> auditorProvider() {
    		return () -> Optional.of(UUID.randomUUID().toString());
    	}
    }
    ```
    
    - 실제 서비스에서는 세션 정보나 스프링 시큐리티의 ID 값 등을 저장하도록 한다고 함.