# 아이템 36. 비트 필드 대신 EnumSet을 사용하라

다음과 같이 비트별 OR를 사용해 여러 상수를 하나의 집합으로 모을 수 있으며 이렇게 만들어진 집합을 비트 필드라 한다.

```java
public class Text {
	public static final int STYLE_BOLD = 1<<0; //1
	****public static final int STYLE_ITALIC = 1<<1; //2
	****public static final int STYLE_UNDERLINE = 1<<2; //4
	****public static final int STYLE_STRIKETHROUGH = 1<<3; //8

	//매개변수 Styles는 0개 이상의 STYLE_상수를 비트별 OR한 값이다.
	public void applyStyles(int syles) {...}
}

text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```

비트 필드를 사용하면 비트별 연산을 사용해 합집합과 교집합 같은 집합 연산을 효울적으로 수행할 수 있다.

**하지만** 비트 필드는 정수 열거 상수의 단점을 그대로 지닌다.

비트 필드값이 그대로 출력되면 단순한 정수 열거 상수를 출력할 때보다 해석하기가 훨씬 어렵다.

비트필드 하나에 녹아있는 모든 원소를 순회하기도 까다롭다

최대 몇 비트가 필요한지를 API작성 시 미리 예측하여 적절한 타입을 선택해야 한다. (API를 수정하지 않고는 비트수를 더 늘릴 수 없기 때문)

**더 나은 대안은 EnumSet사용 하기**

열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해준다

Set인터페이스를 완벽히 구현하며 타입이 안전하고 어떤 set구현체와도 함께 사용할 수 있다.

난해한 작업을 EnumSet이 대부분 처리 해준다.

```java
public class Text {
	public enum Style { BOLD, ITALIC, UNDERLINE, STPIKETHROUGH }

	//어떤 Set을 넘겨도 되나, EnumSet이 가장 좋다.
	public void applyStyles(Set<Style>styles) {...}
}

text.qpplyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```

<aside>
💡 열거할 수 있는 타입을 한데 모아 집합 형태로 사용한다고 해도 비트 필드를 사용할 이유는 없다.
EnumSet클래스가 비트 필드 수준의 명료함과 성능을 제공하고 열거타입의 장점까지 선사하기 때문.
유일한 단점이라면 불변 EnumSet을 만들 수 없지만 명확성과 성능이 조금 희생되더라도 Collextions.unmodifiableSet으로 EnumSet을 감싸 사용할 수 있다.

</aside>
