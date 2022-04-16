# item29. 이왕이면 제네릭 타입으로 만들라
```java
public class Stack {
	private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack() {
    	elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
    
    public void push(Object e) {
    	ensureCapacity();
        elements[size++] = e;
    }
    
    public Obejct pop() {
    	if (siez == 0)
        	throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; //다 쓴 참조 해제
        return result;
    }
    
    public boolean isEmpty() {
    	return size == 0;
    }
    
    private void ensureCapacity() {
    	if (elements.length == size)
        elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

일반 클래스를 제네릭 클래스로 만드는 첫 단계. <br>
클래스 선언에 타입 매개 변수를 추가하는 일. <br>
스택이 담을 원소의 타입 하나만 추가하면 된다. <br>
-> 타입 이름으로는 보통 E를 사용 (아이템 68)
-> Object를 적절한 타입 매개변수로 바꾼다.

```java
public class Stack<E> {
	private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack() {
    	elements = new E[DEFAULT_INITIAL_CAPACITY];
    }
    
    public void push(E e) {
    	ensureCapacity();
        elements[size++] = e;
    }
    
    public E pop() {
    	if (size == 0)
        	throw new EmptyStackException();
        E result = elements[--size];
        elements[size] = null;  // 다 쓴 참조 해재ㅔ
        return result;
    }
    ...// isEmpty와 ensureCapacity 메서드는 그대로
}
```

이렇게 하면 한 가지 오류가 발생한다. <br>
E와 같은 실체화 불가 타입으로는 배열을 만들 수 없음 <br>
<br>
해결책은? <br> <br>

```java
elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
```
제네릭 배열 생성을 금지하는 제약을 대놓고 우회하는 방법. <br>
-> Object 배열을 생성한 다음 제네릭 배열로 형변환. <br>
-> 별로 안전하지 않은 방법 <br>

```java
//배열 elements는 push(E)로 넘어온 E 인스턴스만 담는다.
//따라서 타입 안전성을 보장하지만,
//이 배열의 런타임 타입은 E[]가 아닌 Object[]다!
@SuppressWarnings("unckecked")
public Stack() {
	elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
```

elements 필드의 타입을 E[]에서 Object로 바꾸는 방법.<br>
```java
E result = elements[--size]
```

```java
public E pop() {
	if (size == 0)
    	throw new EmptyStackException();
        
    //push에서 E 타입만 허용하므로 이 형변환은 안전하다.
    @SuppressWarnings("unchecked") E result = (E) elements[--size];
    
    elements[size] = null;  // 다 쓴 참조 해제
    return result;
}
```

첫 번째 방법 <br>
- 가독성이 좋다.
- 코드도 더 짧다
- 현업에서 더 선호하며 자주 사용한다
- 배열의 런타임 타입이 컴파일타임 타입과 달라 힙 오염을 일으킴
(두 번째 방식을 고수하기도 함)

```java
public static void main(String[] args) {
	Stack<String> stack = new Stack<>();
    for (String arg : args)
    	stack.push(arg);
    while (!stack.isEmpty())
    	System.out.println(stack.pop().toUpperCase());
}
```
명령줄 인수들을 역순으로 바꿔 대문자로 출력하는 프로그램. <br>
제네릭 Stack 클래스를 사용하는 모습. <br><br>

Stack에서 꺼낸 원소에서 String의 toUpperCase 메서드를 호출할 때 명시적 형변환을 수행하지 않으며, <br>
(컴파일러에 의해 자동 생성된) 이 형변환이 항상 성공함을 보장. <br>

---
- DelayQueue (타입 매개변수에 제약을 두는 제네릭 타입) <br>
https://runebook.dev/ko/docs/openjdk/java.base/java/util/concurrent/delayqueue <br>
- ClassCastException <br>
https://shs2810.tistory.com/15 <br>
---
>**핵심정리** <br>
클라이언트에서 직접 형변환해야 하는 타입보다 제네릭 타입이 더 안전하고 쓰기 편하다. 그러니 새로운 타입을 설계할 때는 형변환 없이도 사용할 수 있도록 하라. <br>
그렇게 하려면 제네릭 타입으로 만들어야 할 경우가 많다. <br>
기존 타입 중 제네릭이었어야 하는 게 있다면 제네릭 타입으로 변경하자. <br>
기존 클라이언트에는 아무 영향을 주지 않으면서, 새로운 사용자를 훨씬 편하게 해주는 길이다.(아이템 26)
