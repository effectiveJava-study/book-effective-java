# 아이템 28. 배열보다는 리스트를 사용하라

**배열과 제네릭 타입에 차이점1** : 배열은 공변(함께 변한다는 뜻) 제네릭은 불공변

단순 비교로 보면 제네릭이 문제가 있다고 생각 할 수 있지만 문제가 있는것은 배열이다

```java
Object[] objectArray = new Long[1];
objectArray[0] = "나 스트링 타입";
```

```java
List<Object> ol = new ArrayList<Long>();
lo.add("나 스트링 타입");
```

**배열과 제네릭 타입에 차이점2** : 배열은 실체화되고 제네릭은 타입정보가 런타임에 소거된다.

*(실체화...? 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인해서 ArrayStoreException발생!)*

*(소거...? 원소 타입을 컴파일 타임에만 검사하여 런타임에는 알 수 없다.)*

이처럼 결국 배열과 제네릭은 친해지기 어려운 관계다. 배열은 아래와 같이 제네릭 타입, 매개변수화 타입, 타입 매개변수로 사용할 수 없다.

```java
// 배열은 아래와 같이 사용하면 오류가 발생한다.
new List<E>[]; // 제네릭 타입
new List<String>[]; // 매개변수화 타입
new E[]; // 타입 매개변수
```

### 제네릭 배열을 만들지 못하게 막은 이유는 무엇일까?

타입 안전성이 보장되지 않기 때문. 허용하게 된다면 컴파일러가 자동 생성한 형변환 코드에서 런타임 ClassCastException발생. 런타임시점의 형변환 예외가 발생하는것을 막겠다는 제네릭의 취지에 맞지않다.

```java
List<String>[] stringLists = new List<String>[1]; // (1) 
List<Integer> intList = List.of(42);              // (2)
Object[] objects = stringLists;                   // (3)
objects[0] = intList;                             // (4)
String s = stringLists[0].get(0);                 // (5)
```

제네릭 배열을 생성하는**(1)**이 된다고 가정

**(2)**는 원소가 하나인 List<Integer>를 생성

**(3)**은 (1)에서 생성한 List<String>의 배열을 Object배열에 할당 (배열은 공변이는 문제 X)

**(4)**는 (2)에서 생성한 List<Integer>의 인스턴스를 Object배열의 첫 원소로 저장

제네릭은 소거방식으로 구현되어서 성공한다. 

= 런타임 시점에서 타입이 사라지므로 List<Integer>은 List가 되고 List<Integer>[]는 List[]가 된다. 따라서 ArrayStoreException 이 발생하지 않는다.

**(5)**에서 문제가 생긴다. List<String>인스턴스만 담겠다고 선언한 stringLists배열에 다른 타입의 인스턴스가 담겨 있는데 첫 원소를 꺼내려고 한다. 

그리고 이를 String으로 형변환 하는데 이 원소는 integer타입이므로 런타임에 ClassCastException이 발생

**따라서 이러한 일을 방지하려면 제네릭 배열이 생성되지 않도록 (1)번 과정에서 컴파일 오류가 발생해야 함.**

### 실체화 불가 타입

실체화 되지 않아서 런타임에는 컴파일 타임보다 타입정보를 적게 가지는 타입.

E, List<E>, List<String>

소거로 인해 매개변수화 타입 가운데 실체화될 수 있는 타입은 List<?>와Map<?,?>같은 비한정적 와일드 카드 타입뿐.

### 배열로 형 변환시 오류 발생

배열로 형변환할 때 제네릭 배열 생성 오류나 비검사 형변환 경고가 뜨는 경우 대부분은 배열인 E[]대신 컬렉션인 List<E>를 사용하면 해결된다.

코드가 조금 복잡해지고 성능이 살짝 나빠질 수 있지만, 그 대신 타입 안전성과 상호운용성은 좋아진다.

ex)

```java
public class Chooser {
    private final Object[] choiceArray;
    
    public Chooser(Collection choices) {
        this.choiceArray = choices.toArray();
    }
    
    // 이 메서드를 사용하는 곳에서는 매번 형변환이 필요하다.
    // 형변환 오류의 가능성이 있다.
    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceArray[rnd.nextInt(choiceArray.length)];
    }
}
```

```java
public class Chooser<T> {
    private final T[] choiceArray;

    public Chooser(Collection<T> choices) {
        // 오류 발생 incompatible types: java.lang.Object[] cannot be converted to T[]
        this.choiceArray = choices.toArray();
    }

    // choose 메소드는 동일하다.
}
```

오류가 발생한다 incompatible types: java.lang.Object[] cannot be converted to T[]

```java
this.choiceArray = (T[]) choices.toArray();
```

컴파일 오류는 해결 됐지만 Unchecked Cast 경고가 발생한다

T가 무슨 타입인지 알 수 없으니 컴파일러는 이 형변환이 런타임에도 안전한지 보장할 수 없다는 메세지다.

제네릭에서는 원소의 타입 정보가 소거되어 런타임에는 무슨 타입지 알수 없기 때문에 비검사 형변환 경고를 제거하려면 배열 대신 리스트를 쓰면 된다.

```java
class Chooser<T> {
    private final List<T> choiceList;

    public Chooser(Collection<T> choices) {
        this.choiceList = new ArrayList<>(choices);
    }

    public T choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceList.get(rnd.nextInt(choiceList.size()));
    }
}
```

<aside>
💡 배열과 제네릭에는 매우 다른 타입 규칙이 적용된다. 배열은 공변이고 실체화 되는 반면, 제네릭은 불공변이고 타입 정보가 소거된다. 그 결과 배열은 런타임에는 타입이 안전하지만 컴파일 타임에는 그렇지 않다. 제네릭은 반대다. 그래서 둘을 섞어쓰기 쉽지 않다. 둘을 섞어 쓰다가 컴파일 오류나 경고를 만나면, 가장 먼저 배열을 리스트로 대체하는 방법을 적용해 보자.

</aside>
