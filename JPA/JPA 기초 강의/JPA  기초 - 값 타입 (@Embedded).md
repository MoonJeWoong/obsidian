# 기본 타입

기본 타입에는 int, long과 같은 기본 자료형 타입과 Integer, Long과 같은 Wrapping 클래스 타입 그리고 String이 포함된다. 이 기본 타입 데이터들은 항상 값이 복사되어서 전달된다. Integer와 같은 Wrapping 클래스와 String은 공유 자체는 가능하지만 값의 변경이 불가능하다. 그리고 엔티티의 생명주기가 끝나 삭제되면 포함된 기본 타입 데이터들 또한 같이 삭제된다.

# 임베디드 타입

주로 기본 값 타입을 모아서 만들기 때문에 복합 값 타입이라고도 한다.

엔티티 내에 값 객체를 위치시킬 수 있음. 해당 객체는 엔티티는 아니지만 자신이 가지고 있는 속성 값을 이용한 메서드를 가지고 수행할 수도 있다. 이 때 엔티티에 속성 값으로 포함될 값 객체에는 @Embeddable 어노테이션을 선언해줘야 한다. 그리고 엔티티에는 @Embedded어노테이션을 선언해줘야 한다.

~~~ java

@Entity  
@Getter @Setter  
public class Delivery {  
  
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)  
    @Column(name = "delivery_id")  
    private Long id;
  
    @Embedded  
    private Address address; 
}

~~~

중요한 핵심 포인트는 엔티티가 임베디드 타입 객체를 포함하나 임베디드 타입 객체의 속성을 직접 가지나 DB 테이블의 구조는 동일하다는 것이다. 즉 DB의 테이블 구조는 그대로인데 코드를 더 객체지향적으로 구현할 수 있다는 장점을 취할 수 있는 것이다. (코드의 재사용성 향상 + 코드 응집도 향상)

임베디드 타입 객체에는 기본 생성자가 반드시 작성되어야 한다. reflection 사용을 허용하기 위함인데 임베디드 타입 객체는 값 객체로서 불변으로 설계되어야 하는 것이 바람직하므로 기본 생성자를 protected로 선언하고 초기화를 할 수 있는 다른 생성자를 public으로 열어준다.

~~~ java

@Embeddable  
@Getter  
public class Address {  
  
    private String city;  
    private String street;  
    private String zipcode;  
  
    protected Address() {  
    }  
  
    public Address(String city, String street, String zipcode) {  
        this.city = city;  
        this.street = street;  
        this.zipcode = zipcode;  
    }  
}

~~~

임베디드 타입 객체도 내부에서 연관관계를 가질 수 있다. 또 다른 임베디드 타입과 연관관계를 가질 수도 있고, 엔티티와 연관관계를 가질 수도 있다.

한 엔티티 내에서 동일한 임베디드 타입을 상태로 가지는 경우 @AttributeOverrides, @AttributeOverride 어노테이션을 사용한다.

임베디드 타입이 null인 경우 DB에도 null이 저장된다.