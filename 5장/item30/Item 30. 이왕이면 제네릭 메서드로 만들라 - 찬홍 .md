# 이왕이면 제네릭 메서드로 만들라

- 클래스와 마찬가지로, 메서드도 제네릭으로 만들 수 있다.
- Collections의 알고리즘 메서드(binarySearch, sort 등)는 모두 제네릭 이다.
![image](https://user-images.githubusercontent.com/90807343/163665580-10650a41-3668-45ff-8aae-27d7d33af918.png)

### 제네릭 메서드란?
![image](https://user-images.githubusercontent.com/90807343/163665485-f3762c1d-34e3-4490-8c54-09d2f7dcd836.png)

메소드의 선언 부에 적은 제네릭으로 리턴 타입, 파라미터의 타입이 정해지는 메소드

### 제네릭 메서드 작성법

**잘못된 메서드**

---

```java
// 로 타입 사용 - 수용 불가!
public static Set Union(Set s1, Set s2) {
	Set Result = new HashSet(s1);
	result.addAll(s2);
	return result;
}
```

컴파일은 되지만 경고가 두 개 발생한다. 

```java
warning: [unchecked] unchecked call to
....
```

- 이 경고는 간단하게 말하자면 타입을 안전하게 만들라는 이야기이다.
- 메서드 선언에서의 세 집합(입력 2개, 반환 1개)의 원소 타입을 타입 매개변수로 명시하고, 메서드 안에서도 이 타입 매개변수만 사용하게 수정하면 된다.

**제네릭 메서드**

---

```java
public static <E> set<E> union(Set<E> s1, Set<E> s2) {
	Set<E> result = new HashSet<>(s1);
	result.addAll(s2);
	return result;
}
```

- 이 메서드는 경고 없이 컴파일 되며, 타입 안전하고 쓰기도 쉽다.

**제네릭 메서드를 활용하는 간단한 프로그램**

---

```java
public static void main(String[] args) {
	Set<String> guys = Set.of("톰", "딕", "해리");
	Set<String> stooges = Set.of("래리", "모에", "컬리");
	Set<String> aflCio = union(guys, stooges);
	System.out.println(aflCio);
}

출력 : "[모에, 톰, 해리, 래리, 컬리, 딕]"
```

- `union` 메서드는 집합 3개 (입력 2개, 반환 1개)의 타입이 모두 같아야 한다.

<br>

### 불변 객체를 여러 타입으로 활용

- 제네릭은 런타임에 타입 정보가 소거 되므로 하나의 객체를 어떤 타입으로든 매개변수화 할 수 있다.
- 하지만 이렇게 하려면 요청한 타입 매개변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩터리를 만들어야 한다. 이 패턴을 **재네릭 싱글턴 팩터리**라 한다.

<br>

### 제네릭 싱글턴 팩터리란?

제네릭으로 타입 설정 가능한 인스턴스를 만들어 두고, 반환 시에 제네리긍로 받은 타입을 이용해 타입을 결정하는 것

```java
public class GenericFactoryMethod { 
	public static final Set EMPTY_SET = new HashSet(); 

	public static final <T> Set<T> emptySet() { 
		return (Set<T>) EMPTY_SET; 
	} 
}
```

```java
@Test 
public void genericTest() { 
	Set<String> set = GenericFactoryMethod.emptySet(); 
	Set<Integer> set2 = GenericFactoryMethod.emptySet(); 
	Set<Elvis> set3 = GenericFactoryMethod.emptySet(); 

	set.add("ab"); 
	set2.add(123); 
	set3.add(Elvis.INSTANCE); 

	String s = set.toString(); 
	System.out.println("s = " + s); 
}

s = [ab, item3.Elvis@3439f68d, 123]
```

<br>

### 항등함수(identity function)

- 자바 라이브러리의 Function.identity를 사용하는게 편함
- 항등함수 객체는 상태가 없으니 요청할 때 마다 새로 생성하는 것은 낭비다.
- 자바의 제네릭이 실체화 된다면 항등함수를 타입별로 만들어야 하지만, 소거 방식을 사용한 덕에 제네릭 싱글턴 하나면 충분하다.

**제네릭 싱글턴 팩터리 패턴**

---

```java
public static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressWarnings("unchecked") // 비검사 형변환 경고 방지
public static <T> UnaryOperator<T> identiyFunction() {
	return (UnaryOperator<T>) IDENTITY_FN;
```

**제네릭 싱글턴을 사용하는 예**

---

```java
public static void main(String[] args) {
        String[] strings = { "삼베", "대마", "나일론" };
        UnaryOperator<String> sameString = identityFunction();
        for (String s : strings)
            System.out.println(sameString.apply(s));

        Number[] numbers = { 1, 2.0, 3L };
        UnaryOperator<Number> sameNumber = identityFunction();
        for (Number n : numbers)
            System.out.println(sameNumber.apply(n));
    }
```

<br>

## 재귀적 타입 한정(recursive type bound)

- 상대적으로 드물긴 하지만, 자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정할 수 있다.
- 주로 타입의 자연적 순서를 정하는 `Comparable` 인터페이스와 함께 쓰인다.
    
    ```java
    public interface Comparable<T> {
    	int compareTo(T o);
    }
    ```
    
- 재귀적 타입한정을 이용해 상호 비교할 수 있음을 표현한 코드
    
    ```java
    public static <E extends Comparable<E>> E max(Collection<E> c);
    ```
    
    - 타입 한정인 `<E extends Comparable<E>>` 는 **“모든 타입 E는 자신과 비교할 수 있다”** 라고 읽을 수 있다.
- 재귀적 타입 한정은 훨씬 복잡해질 가능성이 있긴 하지만, 다행히 그런 일은 잘 일어나지 않는다.

<br>

### 핵심 정리

---

> 제네릭 타입과 마찬가지로, 클라이언트에서 입력 매개변수와 반환값을 명시적으로 형변환해야 하는 메서드보다 제네릭 메서드가 더 안전하며 사용하기도 쉽다. 
타입과 마찬가지로, 형변환을 해줘야 하는 기존메서드는 제네릭하게 만들자.
기존 클라이언트는 그대로 둔 채 새로운 사용자의 삶을 훨씬 편하게 만들어줄 것이다.
> 

> [Java] 제네릭 메소드(Generic Method)란?
[https://devlog-wjdrbs96.tistory.com/201](https://devlog-wjdrbs96.tistory.com/201)
> 

> 제네릭 싱글턴 팩토리
[https://jake-seo-dev.tistory.com/13?category=909023](https://jake-seo-dev.tistory.com/13?category=909023)
>
