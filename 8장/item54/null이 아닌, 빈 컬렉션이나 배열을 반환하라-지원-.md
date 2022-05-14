# item54. null이 아닌, 빈 컬렉션이나 배열을 반환하라

```java
private final List<Cheese> CheeseInStock = ...;

/**
* @return 매장 안의 모든 치즈 목록을 반환한다.
* 단, 재고가 하나도 없다면 null을 반환한다.
*/

public List<Cheese> getCheeses() {
	return cheeseInStock.isEmpty() ? null
    	: new ArrayList<>(cheeseInStock);
}
```

컬렉션이 비었으면 null을 반환한다. 따라 하지 말 것  <br>

null 상황 처리하는 코드
```java
List<Cheese> Cheeses = shop.getCheeses();
if (cheeses != null && cheeses.contains(Cheese.STILTON))
	System.out.println("좋았어, 바로 그거야.");
```

컬렉션이나 배열 같은 컨테이너가 비었을 때  <br>
null을 반환하는 메서드를 사용할 때면 항시 이와 같은 방어 코드 넣어줘야 한다.  <br>

클라이언트에서 방어 코드를 빼먹으면 오류 발생 가능성  <br>

빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있다.  <br>
빈 컬렉션을 반환하는 전형적은 코드로, 대부분의 상황에서는 이렇게 처리한다고 한다.
```java
public List<cheese> getCheeses() {
	return new ArrayList<>(cheesesInStock);
}
```

사용 패턴에 따라 빈 컬렉션 할당이 성능을 눈에 띄게 떨어뜨릴 수도 있는데,  <br>
해법은 간단.  <br>
-> 매번 똑같은 빈 '불변' 컬렉션을 반환하는 것.  <br>

집합이 필요하면 Collections.emptySet  <br>
맵이 필요하면 Collections.emptyMap  <br>

단, 최적화에 해당하니 꼭 필요할 때만 사용할 것.  <br>
최적화가 필요하다고 판단되면 수정 전과 후의 성능을 축정하여 실제로 성능 개선이 되는지 확인할 것.

```java
public List<cheese> getCheeses() {
	return cheesesInStock.isEmpty() ? Collections.emptyList()
    	: new ArrayList<>(cheesesInStock);
}
```
배열을 쓸 때도 마찬가지로, 절대 null을 반환하지 말고 길이가 0인 배열을 반환하라.

```java
public List<cheese> getCheeses() {
	return cheesesInStock.toArray(new Cheese[0]);
}
```
길이가 0일 수도 있는 배열을 반환하는 올바른 방법. <br>
-> 이 방식이 성능을 떨어뜨릴 것 같다면 길이 0짜리 배열을 미리 선언해두고 매번 그 배열을 반환하면 된다. (길이가 0인 배열은 모두 불변이기 때문)

```java
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
	return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```

최적화 - 빈 배열을 매번 새로 할당하지 않도록 했다.  <br>

이 최적화 버전의 getCheeses는 항상 EMPTY_CHEESE_ARRAY를 인수로 넘겨 toArray를 호출. 따라서 cheeseInStock이 비었을 경우, 언제나 EMTPY_CHEESE_ARRAY를 반환하게 된다.  <br>

단순히 성능 개선 목적이라면 toArray에 넘기는 배열을 미리 할당하는 건 추천하지 않는다. 오히려 성능이 떨어진다고 한다.  <br>

ex)나쁜 예시 : 배열 미리 할당하면 성능 나빠짐
```java
return cheeseInStock.toArray(new Cheese[cheesesInStock.size()];
```

---

>**핵심정리**  <br>
null이 아닌, 빈 배열이나 컬렉션을 반환하라.  <br>
null을 반환하는 API는 사용하기 어렵고 오류 처리 코드도 늘어난다.  <br>
그렇다고 성능이 좋은 것도 아니다.
