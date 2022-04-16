# 아이템30. 이왕이면 제네릭 메서드로 만들라

클래스와 마찬가지로, 메서드도 제네릭으로 만들 수 있다. 매개변수화 타입을 받는 정적 유틸리티 메서드는 보통 제네릭이다.

제네릭 메서드 작성법은 제네릭 타입 작성법과 비슷하다.

```java
public static Set union(Set s1, Set s2) {
   Set result = new HashSet(s1);
   result.addAll(s2);
   return result;
}
```

컴파일은 되지만 경고가 두개 발생하게 된다

`Set result = new HashSet(s1);`

`result.addAll(s2);`

경고를 없애려면 이 메서드를 타입 안전하게 만들어야 한다. 메서드 선언에서의 세 집합의 원소 타입을 타입매개변수로 명시하고, 메서드 안에서도 이 타입 매개변수만 사용하게 수정하면 된다.

```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
   Set<E> result = new HashSet<>(s1);
   result.addAll(s2);
   return result;
}
```

- 때때로 불변 객체를 여러 타입으로 활용할 수 있게 만들어야 할 때가 있는데 제네릭은 런타임에 타입 정보가 소거 되므로 하나의 객체를 어떤 타입으로든 매개변수화 할 수 있다.
- 하지만 이렇게 하려면 요청한 타입 매개변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩터리를 만들어야한다.
- 이 패턴을 제네릭 싱글턴 팩터리라 한다.

```java
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

    @SuppressWarnings("unchecked")
    public static <T> UnaryOperator<T> identityFunction() {
        return (UnaryOperator<T>) IDENTITY_FN;
    }
```

- DENTITY_FN을 `UnaryOperator<T>`로 형변환하면 비검사 형변환 경고가 발생한다. T가 어떤 타입이든 `UnaryOperator<Object>`는 `UnaryOperator<T>`가 아니기때문이다.
- 하지만 항등함수란 입력 값을 수정 없이 그대로 반환하는 특별한 함수이므로, T가 어떤 타입이든 `UnaryOperator<T>`를 사용해도 타입 안전하다.

```java
public static void main(String[] args) {
        String[] strings = {"삼베", "대마", "나일론"};
        UnaryOperator<String> sameString = identityFunction();
        for (String s : strings) {
            System.out.println(sameString.apply(s));
        }

        Number[] numbers = {1, 2.0, 3L};
        UnaryOperator<Number> sameNumber = identityFunction();
        for (Number n : numbers) {
            System.out.println(sameNumber.apply(n));
        }
    }
```

### 재귀적 타입 한정

자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정할 수 있다.

주로  타입의 자연적 순서를 정하는 Comparable인터페이스와 함께 쓰인다.

```java
public interface Comparable<T> {
    int compareTo(T o);
}
```

```java
public static <E extends Comparable<E>> E max(Collection<E> c);
```

```java
public static <E extends Comparable<E>> E max(Collection<E> collection) {
        if (collection.isEmpty()) {
            throw new IllegalArgumentException("컬렉션이 비어 있습니다.");
        }

        E result = null;
        for (E e : collection) {
            if (result == null || e.compareTo(result) > 0) {
                result = Objects.requireNonNull(e);
            }
        }
        return result;
    }
```

위 메서드에서는 빈 컬렉션인 경우 IllegalArgumentException을 던지니, Optional<E>를 반환하도록 하는것이 더 낫다.

<aside>
💡 제네릭 타입과 마찬가지로, 클라이언트에서 입력 매개변수와 반환값을 명시적으로 형변환 해야 하는 메서드보다 제네릭 메서드가 더 안전하며 사용하기도 쉽다.
타입과 마찬가지로, 메서드도 형변환 없이 사용할 수 있는 편이 좋으며, 많은 경우 그렇게 하려면 제네릭 메서드가 되어야 한다. 역시 타입과 마찬가지로, 형변환을 해줘야 하는 기존 메서드는 제네릭하게 만들자.

</aside>
