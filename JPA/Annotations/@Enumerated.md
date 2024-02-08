
엔티티의 속성으로 Enum을 사용하기 위해 필요한 어노테이션으로 ORDINAL과 STRING 옵션 중에 선택할 수 있다. 일반적으로 STRING 옵션을 사용한다.

~~~ java

public enum EnumType {
    /** Persist enumerated type property or field as an integer. */
    ORDINAL,

    /** Persist enumerated type property or field as a string. */
    STRING

~~~
