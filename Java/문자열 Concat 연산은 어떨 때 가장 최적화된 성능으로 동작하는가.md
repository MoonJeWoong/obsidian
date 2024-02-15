### String + String, StringBuilder

String 은 기본적으로 불변이기 때문에 2개의 String 타입을 이어 붙이는 '+'연산을 수행할 때, 새로운 변수를 생성하고 데이터를 이어 붙입니다.

~~~ java
@Test  
void test() {  
    String str = "a" + "b" + "c";  
}
~~~

JDK1.8

~~~ java
L0
    LINENUMBER 6 L0
    LDC "abc"
    ASTORE 1
~~~

JDK9 이후

~~~ java
L0
    LINENUMBER 6 L0
    LDC "abc"
    ASTORE 1
~~~

결론 : JDK 버전에 상관없이 한 줄에 명시되는 concat 명령은 컴파일 타임에 최적화됩니다.



그렇다면 한 줄이 아니라 여러 줄에 걸쳐 진행되는 concat 연산은 어떻게 컴파일될까요?

~~~ java
@Test  
void test() {  
    String str = "a";  
    str +=  "b";  
    str +=  "c";  
}
~~~


JDK 1.8
~~~ java
L0
    LINENUMBER 6 L0
    LDC "a"
    ASTORE 1
L1
    LINENUMBER 7 L1
    NEW java/lang/StringBuilder
    DUP
    INVOKESPECIAL java/lang/StringBuilder.<init> ()V
    ALOAD 1
    INVOKEVIRTUAL java/lang/StringBuilder.append (Ljava/lang/String;)Ljava/lang/StringBuilder;
    LDC "b"
    INVOKEVIRTUAL java/lang/StringBuilder.append (Ljava/lang/String;)Ljava/lang/StringBuilder;
    INVOKEVIRTUAL java/lang/StringBuilder.toString ()Ljava/lang/String;
    ASTORE 1
L2
    LINENUMBER 8 L2
    NEW java/lang/StringBuilder
    DUP
    INVOKESPECIAL java/lang/StringBuilder.<init> ()V
    ALOAD 1
    INVOKEVIRTUAL java/lang/StringBuilder.append (Ljava/lang/String;)Ljava/lang/StringBuilder;
    LDC "c"
    INVOKEVIRTUAL java/lang/StringBuilder.append (Ljava/lang/String;)Ljava/lang/StringBuilder;
    INVOKEVIRTUAL java/lang/StringBuilder.toString ()Ljava/lang/String;
    ASTORE 1
~~~

처음 문자열을 초기화하는 코드는 이전과 동일하게 컴파일 되었지만 이후 진행되는 concat연산에 대해서는 StringBuilder를 초기화하고 기존 문자열을 append 하고 새로운 문자열도 append를 해서 연산을 수행하도록 컴파일 된 것을 확인할 수 있습니다.

만약 반복문을 돌면서 1000번 문자열을 concat 한다면 매 연산마다 StringBuilder가 새로 생성될 것이므로 비효율적인 연산이라고 얘기할 수 있습니다. 따라서 반복문을 통해 여러번 문자열을 concat 하는 경우에는 "+" 연산자 보다는 StringBuilder를 처음부터 생성해서 append 하도록 하는 것이 더 효율적일 것입니다.


JDK9 이후

~~~ java
L0
    LINENUMBER 6 L0
    LDC "a"
    ASTORE 1
L1
    LINENUMBER 7 L1
    ALOAD 1
    INVOKEDYNAMIC makeConcatWithConstants(Ljava/lang/String;)Ljava/lang/String; [
      // handle kind 0x6 : INVOKESTATIC
      java/lang/invoke/StringConcatFactory.makeConcatWithConstants(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/String;[Ljava/lang/Object;)Ljava/lang/invoke/CallSite;
      // arguments:
      "\u0001b"
    ]
    ASTORE 1
L2
    LINENUMBER 8 L2
    ALOAD 1
    INVOKEDYNAMIC makeConcatWithConstants(Ljava/lang/String;)Ljava/lang/String; [
      // handle kind 0x6 : INVOKESTATIC
      java/lang/invoke/StringConcatFactory.makeConcatWithConstants(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/String;[Ljava/lang/Object;)Ljava/lang/invoke/CallSite;
      // arguments:
      "\u0001c"
    ]
    ASTORE 1
~~~

JDK 9 이후부터는 StringBuilder가 아니라 makeConcatWithConstants라는 메서드를 사용하는 것으로 최적화되어 컴파일되는 것을 확인할 수 있습니다. 


### StringBuilder vs makeConcatWithConstants 성능 비교 테스트

~~~ java
@Test  
void test() {  
    int iterTime = 300_000;  
    long beforeTime = System.currentTimeMillis(); // 코드 실행 전에 시간 받아오기  
  
    String str = "a";  
    for (int i=0; i < iterTime; i++) {  
        str += "c";  
    }  
  
    long afterTime = System.currentTimeMillis(); // 코드 실행 후에 시간 받아오기  
    long diffTime = (afterTime - beforeTime); // 두 개의 실행 시간  
    System.out.println("실행 시간(ms): " + diffTime); // 세컨드(초 단위 변환)  
}
~~~


JDK1.8

~~~
실행 시간(ms): 3234     // iter_time = 100_000
실행 시간(ms): 6009     // iter_time = 200_000
실행 시간(ms): 11075    // iter_time = 300_000
~~~

JDK9 이후

~~~ 
실행 시간(ms): 467     // iter_time = 100_000
실행 시간(ms): 1545    // iter_time = 200_000
실행 시간(ms): 3295    // iter_time = 300_000
~~~

반복 횟수가 늘어나면 늘어날수록 성능 차이가 극명하게 벌어지는 것을 확인할 수 있습니다.

여기서 알 수 있는 결론은 매번 StringBuilder를 생성해서 concat 연산을 수행하는 것의 오버헤드가 크게 발생한다는 것입니다. 그렇다면 처음부터 StringBuilder를 생성해서 사용하도록 변경하면 코드의 성능 개선이 어떻게 이뤄질까요?

~~~ java
@Test  
    void test() {  
        int iterTime = 100_000;  
  
        long beforeTime = System.currentTimeMillis(); // 코드 실행 전에 시간 받아오기  
//        기존 코드
//        String str = "a";  
//        for (int i=0; i < iterTime; i++) {  
//            str += "c";  
//        }  
  
        StringBuilder sb = new StringBuilder("a");  
        for (int i=0; i < iterTime; i++) {  
            sb.append("c");  
        }  
  
        long afterTime = System.currentTimeMillis(); // 코드 실행 후에 시간 받아오기  
        long diffTime = (afterTime - beforeTime); // 두 개의 실행 시간  
        System.out.println("실행 시간(ms): " + diffTime); // 세컨드(초 단위 변환)  
    }
~~~

~~~
실행 시간(ms): 3     // iter_time = 100_000
실행 시간(ms): 4     // iter_time = 200_000
실행 시간(ms): 5     // iter_time = 300_000
~~~

"+" 연산을 통해 concat을 수행했을 때보다 극적인 성능 개선을 확인할 수 있습니다.

결론 : Concat 작업을 수행할 때는 StringBuilder를 명시적으로 생성해서 사용하도록 하는 것이 "+" 연산을 사용할 때 compiler에서 최적화를 해주는 것보다 더 우월한 성능으로 동작한다는 것을 알 수 있습니다.

---
Reference

- [문자열 concat 시 발생하는 InvokeDynamic 관련 바이트 코드 변환 설명 ](https://www.baeldung.com/java-string-concatenation-invoke-dynamic)