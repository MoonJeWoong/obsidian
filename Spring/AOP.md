
콩하나의 AOP
- AOP란 Aspect Oriented Programming의 약자로, 관점 지향 프로그래밍을 의미한다.
- AOP는 핵심적인 비즈니스 로직으로부터 횡단 관심사를 분리하는 것에 목적을 둔다.
- 즉, AOP란 부가적인 기능들을 따로 관리하는 것을 의미한다.

횡단 관심사 : 애플리케이션에서 코드가 중복되고, 강력하게 결합되어 있어 다른 로직과 분리할 수 없는 애플리케이션 로직
ex) 로깅, 보안, 트랜잭션, 메서드 실행시간 측정 등...

### AOP의 장점
- 전체 애플리케이션에 산재되어 있는 횡단 관심사 로직들을 한데로 응집시킨다.
	- 코드의 중복을 제거하고 AOP 객체의 응집도를 높인다.
- 횡단 관심사 로직을 핵심 비즈니스 로직으로부터 분리하여 각 로직의 가독성을 향상시킬 수 있다. 

결과적으로 AOP는 객체지향적으로 코드를 구현할 수 있도록 도와주며 유지보수성을 증대시킨다.


### AOP 용어
- Tartget : 부가기능을 부여할 대상
- Advice : target에게 부여할 부가기능을 구현한 모듈
	- 무슨 기능을 적용할 것인지에 대한 정보를 정의
	- 어떤 시점에 적용할 것인지에 대한 정보를 정의
- Join Point : 어드바이스가 적용될 수 있는 위치
	- <mark style="background: #FFF3A3A6;">메서드 호출, 필드 값 변경 등이 JoinPoint에 해당한다.</mark>
	- 스프링에서는 프록시를 이용해서 AOP를 구현하기 때문에 메서드 호출에 대한 JoinPoint만 지원한다.
- Pointcut : 실제 Advice를 적용할 Join Point를 나타냄.
	- 정규 표현식이나 AspectJ 문법을 활용해서 PointCut을 정의할 수 있음.
- Advisor : pointcut과 advice를 하나씩 갖고 있는 오브젝트
- Weaving : <mark style="background: #FFF3A3A6;">join point</mark>에 advice를 적용하는 것
	- 스프링 AOP에서는 Runtime Weaving을 사용한다.



### 스프링에서 AOP의 동작 방식

- 스프링에서는 bean을 생성하고 스프링 컨테이너에 등록하기 전에 BeanPostProcessor(빈 후처리기)를 사용해 조작을 해준다.
- BeanPostProcessor에서는 다음과 같은 작업들을 처리한다.
	1. 생성된 빈을 전달받는다.
	2. 빈 교체 대상을 확인한다.
		- 어드바이저 내의 포인트 컷을 이용해서 전달받은 빈이 프록시 적용 대상인지 확인한다.
	3. 빈 교체 대상이 확인되면 이를 조작 또는 교체한다.
		- 프록시 생성 대상 빈들을 프록시 객체로 변환시켜 준다.
	4. 교체하거나 전달받은 빈을 반환한다.
		- 프록시로 변환된 빈이라면 프록시 객체를, 아니라면 그냥 빈을 그대로 반환한다.
	5. 반환된 빈 객체를 컨테이너에 등록한다.

![[Pasted image 20231030174044.png]]


### 프록시

원본 객체를 감싼 새로운 객체로, 스프링 AOP에서 부가기능을 추가할 때 사용하는 패턴




### @AspectJ 어노테이션을 활용한 AOP 구현

``` java
execution(int minus(int, int))  // 반환타입 int, 두 개의 int 타입 파라미터를 받는 minus 메서드를 표시
execution(* minus(int, int))    // 반환 타입은 상관 없음.
execution(* minus(..))          // 반환 타입과 매개변수 모두 상관 없음
execution(* m*s(..))            // 반환 타입과 매개변수 상관 없이 메서드 명이 m으로 시작해서 s로 끝나는 메서드를 표시
execution(* *(..))              // 모든 메서드
```

![[Pasted image 20231030175914.png]]

