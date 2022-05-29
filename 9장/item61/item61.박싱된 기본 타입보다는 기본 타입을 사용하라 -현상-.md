# 아이템 61

# 박싱된 기본 타입보다는 기본 타입을 사용하라

### 자바의 데이터 타입 2가지

- 기본 타입 : int, double, boolean 등
- 참조 타입 : String, List 등

각각의 기본타입에 대응하는 참조 타입이 있다 이를 박싱된 기본 타입이라고 한다.

(Interger, Double, Boolean 등) 오토 박싱과 오토 언박싱 덕분에 편하게 구분 없이 사용할 수 있다.

그래도 분명히 **차이**는 존재한다.

### 첫 번째 차이

기본 타입은 값만 가지고 있지만, 박싱된 기본 타입은 값에 더해 식별성 이라는 속성도 갖는다.

(각 객체는 언제나 자신만의 주소값을 가진 고유의 존재)

두개의 박싱된 기본 타입은 값(value)이 같아도 결국 서로 다르다는 것을 의미한다.

```java
Comparator<Integer> naturalOrder = (i,j) -> (i < j) ? -1 : (i == j ? 0 : 1);

naturalOrder.compare(new Integer(42), new Integer(42));
```

위 코드를 보면 두 Integer의 값이 같아서 0이 출력돼야 하지만 실제로는 1이 출력된다.

박싱된 기본 타입간에 ‘==’ 연산은 각 객체의 주소를 비교하기 떄문이다.

```java
Comparator<Interger> naturalOrder = (iBoxed, JBoxed) -> {
	int i = iBoxed, j = jBoxed; //오토 박싱
	return i < j ? - 1 : (i == j ? 0 : 1);
}
```

위 코드 처럼 박싱된 매개변수의 갑ㅅ을 기본 타입으로 저장 후 수행하면 잘 된다고 한다.

### 두 번째 차이

기본 타입의 값은 언제나 유효한 값이 들어간다. 하지만 박싱된 기본 타입은 참조 타입이기 때문에 유효하지 않은 값, 즉 null이 들어갈 수 있다.

```java
public class Unbelievable {
    static Integer i;

    public static void main(String[] args) {
        if (i == 42)
            System.out.println("믿을 수 없군!");
    }
}
```

위 코드는 아무것도 출력되지 않게 만든 코드이기도 하지만, 그전에 에러가 발생한다 

( i == 42 ) 를 할 때, null 참조를 언박싱하게 되면서 NPE가 터진다.

해결 방법은 첫 번째 방법처럼 i의 타입을 Interger에서 int로 바꿔주면 해결이된다.

### 세 번째 차이

기본 타입보다 박싱된 기본 타입을 사용하면 시간과 메모리 사용면에서 매우 불리하다.

```java
public static void main(String[] args) {
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++) {
        sum += i;
    }
    System.out.println(sum);
}
```

위 코드에서는 sum을 박싱 타입으로 선언하여 박싱과 언박싱이 반복해서 일어나 체감이 될 정도로 성능이 느린 코드이다.

<aside>
💡 핵심 정리 : 
기본 타입이 간단하고 빠르고 신경을 덜 써도 되니, 
웬만하면 기본 타입을 사용하는 것이 좋다.

</aside>
