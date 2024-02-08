
### @Table


### @GeneratedValue

일반적으로 @Id 어노테이션을 선언해서 엔티티의 기본키(PK)를 지정한다. 이때 자동으로 생성되는 값을 Id로 사용하기 위해 @GeneratedValue 어노테이션과 함께 사용한다.

- strategy 타입 
~~~ java

@GeneratedValue(strategy = GenerationType.Auto)

// DB에서 관리하는 AutoIncrement 값을 토대로 id 값을 관리한다.
@GeneratedValue(strategy = GenerationType.IDENTITY)

// DB에서 관리하는 Sequence object를 사용해서 id 값을 관리한다.
@SequenceGenerator(
	name = "MEMBER_SEQ_GENERATOR",
	sequenceName = "MEMBER_SEQ",  // 매핑할 DB 시퀀스의 이름
	initialValue = 1,  // 한번에 조회해올 id 개수
	allocationSize = 1
)
@GeneratedValue(strategy = GenerationType.SEQUENCE,
				generator = "MEMBER_SEQ_GENERATOR"
)

// 키 생성 전용 테이블을 만들어서 시퀀스 전략을 흉내내는 방법
// 시퀀스 전략과 동일하게 @SequenceGenerator가 필요하다.
@SequenceGenerator(
	name = "MEMBER_SEQ_GENERATOR",
	table = "MY_SEQUENCE",  // DB 테이블 이름
	pkColumnValue = "MEMBER_SEQ",
	allocationSize = 1
)
@GeneratedValue(strategy = GenerationType.TABLE
				generator = "MEMBER_SEQ_GENERATOR"
)

~~~

