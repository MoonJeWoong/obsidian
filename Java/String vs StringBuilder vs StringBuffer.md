

### String

자바에서 String은 참조 타입에 해당합니다. 그래서 변수에 값을 저장하면 그 값을 그대로 저장하는 것이 아니라 힙 메모리 영역에 생성된 객체의 번지 수를 저장하게 됩니다. 

String은 생성하는 방식은 크게 2가지로 나뉘는데 그에 따라 힙 메모리 영역에서 생성되고 관리되는 방식이 달라집니다. 

먼저 문자열 리터럴로 String 변수에 값을 할당하는 경우 힙 메모리 영역 내의 문자열 상수풀에 객체가 생성되고 그 번지수가 저장됩니다. 이후 동일한 문자열 리터럴이 할당되어야 한다면 새로운 객체를 생성하지 않고 기존에 생성되어 있던 문자열 객체의 번지수를 저장합니다. 

반면 new 키워드를 통해 String 객체를 직접 생성해준다면 동일한 문자열 값을 가지더라도 다른 객체로 생성되게 됩니다. 

~~~ java

/**  
 * 동일한 문자 리터럴은 상수풀에 캐싱되어 동일하게 사용된다.  
 */@Test  
void 문자_리터럴을_사용해_생성된_문자열들은_리터럴이_같으면_동일한_객체를_가리키는_참조값을_갖는다() {  
    // given  
    String str1 = "홍길동";  
    String str2 = "홍길동";  
  
    // when & then  
    assertThat(str1 == str2).isTrue();  
}  
  
@Test  
void new_키워드를_통해_생성된_문자열들은_서로_다른_객체로서_다른_참조값을_갖는다() {  
    // given  
    String str1 = "홍길동";  
    String str2 = new String("홍길동");  
  
    // when & then  
    assertThat(str1 != str2).isTrue();  
}

~~~


### StringBuilder , StringBuffer

~~~ java
public final class StringBuilder  
    extends AbstractStringBuilder  
    implements java.io.Serializable, Comparable<StringBuilder>, CharSequence  
{...}


public final class StringBuffer  
    extends AbstractStringBuilder  
    implements Serializable, Comparable<StringBuffer>, CharSequence  
{...}

~~~

StringBuilder와 StringBuffer 모두 AbstractStringBuilder를 상속해서 구현하고 있습니다. 구현 코드를 살펴보면 수행하는 작업 내용 자체는 차이가 없음을 확인할 수 있습니다.

~~~ java
// StringBuilder
@Override  
public StringBuilder append(char[] str) {  
    super.append(str);  
    return this;
}


// StringBuffer
@Override  
@IntrinsicCandidate  
public synchronized StringBuffer append(String str) {  
    toStringCache = null;  
    super.append(str);  
    return this;}
~~~

차이점이 있다면 StringBuffer에는 대부분의 메서드가 Synchronized로 선언되어 있다는 것입니다. 따라서 멀티 스레드 환경 등에서 사용되어야 하는 경우, ThreadSafe를 보장하기 위해 StringBuffer를 사용하고 그 이외에는 StringBuilder를 사용하면 됩니다.

