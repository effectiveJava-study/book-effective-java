# 상속보다는 컴포지션을 사용하라

'상속'   
: 클래스가 다른 클래스를확장하는 구현 상속   

상위 클래스는 릴리스마다 내부 구현이 달라질 수 있으며,   
그 여파로 코드 한 줄 건드리지 않은 하위 클래스가 오동작 할 수 있다.   
-> "메서드 호출과 달리 상속은 캡슐활르 깨뜨린다."

```java
public class InstrumentHashSet<E> extends HashSet<E>{
  //추가된 원소의 수 
  private int addCount = 0;

  public InstrumentHashSet(int initCap, float loadFactor) {
      super(initCap, loadFactor);
  }

  @Override public boolean add(E e) {
      addCount ++;
      return super.add(e);
  }

  @Override public boolean addAll(Collection<? extends E> c) {
      addCOunt += c.size();
      return super.addAll(c);
  }

  public int AddCount() {
      return addCount;
  }

}
```


```java
InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
s.addAll(List.of("틱", "탁탁", "펑"));
```

```
addCount에 3을 더한 후 HashSet의 addAll 호출,
HashSet의 addAll은 각 원소를 add 메서드를 호출해서 추가하는데,
이때 불리는 add는 InstrumentedHashSet에서 재정의한 메서드.
-> addCount에 값이 중복해서 더해져, 최종값이 6으로 늘어난 것.
addAll로 추가한 원소 하나당 2씩 증가.
```

상위 클래스에 새 메서드를 추가하는 방법은?   
운 없게도 하필 하위 클래스에 추가한 메서드와    
시그니처가 같고 반환 타입은 다르다면 컴파일조차 되지 않는다.

묘안.   
기존 클래스를 확장하는 대신,    
새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 하자.

"컴포지션"   
: 기존 클래스가 새로운 클래스의 구성요소로 쓰인다는 뜻.에서    
이러한 설계를 컴포지션(composition; 구성)이라 함.

새 클래스의 인스턴스 메서드들은    
(private 필드로 참조하는) 기존 클래스의 대응하는 메서드를 호출해 그 결과를 반환

이 방식을 전달(forwarding)이라 한다.

새 클래스의 메서드들을 전달 메서드(forwarding method)라 부른다.

---
위의 코드를 하나는 집합 클래스 자신,   
다른 하나는 전달 메스만으로 이뤄진 재사용 가능한 전달 클래스 예시.

```java
public class InstrumentedSet<E> extends ForwardingSet<E> {
	private int addCount = 0;

	public InstrumentedSet (Set<E> s){
		super(s);
	}

	@Override public boolean add(E e) {
		addCount++;
		return super.add(e);
	}

	@Override public boolean addAll(Collection<? extends E> c) {
		addCount += c.size();
		return super.addAll(c);
	}

	public int getAddCount() {
		return addCount;
	}
}
```

```java
public class ForwardingSet<E> implements Set<E> {
	private final Set<E> s;
	public ForwardingSet(Set<E> s) { this.s = s; }

	public void claer()					{ s.clear(); }
	public boolean contains(Object o)	{ return s.contains(o); }
	public boolean isEmpty()			{ return s.isEmpty(); }
	public int size()					{ return s.size(); }
	public Iterator<E> iterator()		{ return s.iterator(); }
	public boolean add(E e)				{ return s.add(e); }
	public boolean remove(Object o)		{ return s.remove(o); }
	public boolean containsAll(Collection<?> c) { return s.containsAll(c); }
    public boolean addAll(Collection<?> c) { return s.addAll(c); }
	public boolean removeAll(Collection<?> c) { return s.removeAll(c); }
    public boolean retainAll(Collection<?> c) { return s.retainAll(c); }
	public Object[] toArray()			{ return s.toArray(); }
	public <T> T[] toArray(T[] a)		{ return s.toArray(a); }
	@Override public boolean equals(Object o) { return s.equals(o); }
	@Override public int hashCode()		{ return s.hashCode(); }
	@Override public String toString()	{ return s.toString(); }
}
```

HashSet의 모든 기능을 정의한 Set 인터페이스를 활용해 설계.
-> 견고하고 유연하다.

구체적으로 Set 인터페이스를 구현했고,
Set의 인스턴스를 인수로 받는 생성자를 하나 제공.

>임의의 Set 계측 기능을 덧씌워 새로운 Set으로 만드는 것이 이 클래스의 핵심.

- 상속 방식 :
구체 클래스 각각을 따로 확장해야 하며,
지원하고 싶은 상위 클래스의 생성자 각각에 대응하는 생성자를 별도로 정의

- 래퍼 클래스 :
다른 Set 인스턴스를 감싸고(wrap) 있다는 뜻에서 InstrumentedSet 같은 클래스

- 데코레이터 패던(Decorator pattern) :
다른 Set에 계측 기능을 덧씌운다는 뜻

- 위임(delegation) :
컴포지션과 전달의 조합의 넓은 의미로 부름
(단, 래퍼 객체가 내부 객체에 자기 자신의 참조를 넘기는 경우만 해당)

래퍼 클래스는 단점이 거의 없으나,
콜백(callback) 프레임워크와는 어울리지 않다는 점만 주의.

- 콜백 프레임워크 :    
자기 자신의 참조를 다른 객체에 넘겨서 다음 호출(콜백) 때 사용하도록 함.

- SELF 문제 :   
자신(this)의 참조를 넘기고, 콜백 때는 래퍼가 아닌 내부 객체 호출

> 상속은 반드시 하위 클래스가 상위 클래스의 '진짜' 하위 타입인 상황에서만 쓰여야 한다.   
클래스가 B가 클래스 A와 is-a 관계일 때만 클래스 A를 상속해야 함.   
"B가 정말 A인가?"

'아니다' 일 경우, A를 private 인스턴스로 두고,   
A와는 다른 API를 제공해야 하는 상황.   
즉, A는 B의 필수 구성요소 x 구현 방법 중 하나 o


> 컴포지션으로는 결함을 술김는 새로운 API를 설계할 수 있지만,   
상속은 상위 클래스의 API를 '결함까지도' 그대로 승계


---
Collection<? extends E> c
https://installed.tistory.com/entry/%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0-ArrayList-%EC%A0%9C%EB%84%A4%EB%A6%AD-%ED%81%B4%EB%9E%98%EC%8A%A4-%ED%99%95%EC%A0%95for-%EB%AC%B8#:~:text=1.%20ArrayList%20%28Collection%3C%3F%20extends%20E%3E%20c%29%20-%20Collection%3C%3F,c%20%3A%20E%ED%83%80%EC%9E%85%EC%9D%98%20%ED%81%B4%EB%9E%98%EC%8A%A4%EC%9D%B4%EA%B1%B0%EB%82%98%20E%ED%83%80%EC%9E%85%EC%9D%98%20%EB%B6%80%EB%AA%A8%ED%81%B4%EB%9E%98%EC%8A%A4%EA%B0%80%20%EC%A0%9C%EB%84%A4%EB%A6%AD%ED%83%80%EC%9E%85%EC%9C%BC%EB%A1%9C%20%EC%A7%80%EC%A0%95%EB%90%A8

제네릭 / E타입 / T타입
https://velog.io/@ayoung0073/java-generic

프레임워크, 콜백
https://purple-wood-lights.tistory.com/6

스택, 벡터
https://doorrock.tistory.com/5

