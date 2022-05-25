# 전통적인 for 문보다는 for-each 문을 사용하라

- 스트림이 제격인 작업이 있고 반복이 제격인 작업이 있다.

**컬렉션 순회하기 - 더 나은 방법이 있다.**

---

```java
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
    Element e = i.next();
    ... // 으로 무언가를 한다.
}
```

<br>

**배열 순회하기 - 더 나은 방법이 있다.**

---

```java
for (int i = 0; i < a.length; i++) {
    ... // a[i]로 무언가를 한다.
}
```

- 이 관용구들은 `while` 문 보다는 낫지만 가장 좋은 방법은 아니다.
- 반복자와 인덱스 변수는 모두 코드를 지저분하게 할 뿐 우리에게 진짜 필요한 건 원소들 뿐이다.
- 더군다나 이처럼 쓰이는 요소 종류가 늘어나면 오류가 생길 가능성이 높아진다.
- 이상의 문제는 `for-each` 문을 사용하면 모두 해결된다.


<br>

## for-each 문

- 정식 이름은 “향상된 for문 (enhanced **for** statement)”이다.
- 반복자와 인덱스 변수를 사용하지 않으니 코드가 깔끔해지고 오류가 날 일도 없다.
- 하나의 관용구로 컬렉션과 배열을 모두 처리할 수 있어서 어떤 컨테이너를 다루는지는 신경쓰지 않아도 된다.


<br>

**컬렉션과 배열을 순회하는 올바른 관용구**

---

```java
for (Element e : elements) {
    ... // e로 무언가를 한다.
}
```

- 여기서 콜론(:)은 “안의(in)” 라고 읽으면 된다.
- 반복 대상이 컬렉션이든 배열이든, `for-each` 문을 사용해도 속도는 그대로다.
- `for-each` 문이 만들어내는 코드는 사람이 손으로 최적화한 것과 사실상 같기 때문이다.


<br>

### 컬렉션을 중첩해 순회해야 한다면 `for-each` 문의 이점이 더욱 커진다.

**버그를 찾아보자**

---

```java
enum Suit { CLUB, DIAMOND, HEART, SPADE }
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT,
    NINE, TEN, JACK, QUEEN, KING }
...

static Collection<Suit> suits = Arrays.asList(Suit.values());
static Collection<Rank> ranks = Arrays.asList(Rank.values());

List<Card> deck = new ArrayList<>();
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
    for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
        deck.add(new Card(i.next(), j.next()));
```

- 바깥 컬렉션(suits)의 반복자에서 `next` 메서드가 너무 많이 불린다는 것이다.
- 마지막 줄의 `i.next()`를 주목하자
- 이 next()는 ‘카드(Suit) 하나당’ 한 번씩만 불려야 하는데, 안쪽 반복문에서 호출되는 바람에 ‘숫자(Rank) 하나당’ 한 번씩 불리고 있다.
- 그래서 카드가 바닥나면 반복문에서 `NoSuchElementException`을 던진다.
- 정말 운이 나빠서 바깥 컬렉션의 크기가 안쪽 컬렉션 크기의 배수라면(예컨대 같은 컬렉션일 때 이럴 수 있다.) 이 반복문은 예외를 던지지 않고 종료한다.
- 물론 우리가 원하는 일을 수행하지 않은 채 말이다.


<br>

**같은 버그, 다른 증상!**

---

```java
enum Face { ONE, TWO, THREE, FOUR, FIVE, SIZ }
...
Collection<Face> faces = EnumSet.allOf(Face.class);

for(Iterator<Face> i = faces.iterator(); i.hasNext(); )
	for(Iterator<Face> j = faces.iterator(); j.hasNext(); )
		System.out.println(i.next() + " " + j.next());
```

- 이 프로그램은 예외를 던지진 않지만, 가능한 조합을(”ONE ONE” 부터 “SIX SIX”까지) 단 여섯 쌍만 출력하고 끝나버린다.
- 이 문제를 해결하려면 바깥 반복문에 바깥 원소를 저장하는 변수를 하나 추가해야 한다.


<br>

**문제는 고쳤지만 보기 좋진 않다. 더 나은 방법이 있다!**

---

```java
for(Iterator<Face> i = faces.iterator(); i.hasNext(); ) {
	Face face = i.next();
	for(Iterator<Face> j = faces.iterator(); j.hasNext(); )
		System.out.println(face + " " + j.next());
}
```

- `for-each` 문을 중첩하는 것으로 이 문제는 간단히 해결된다.


<br>

**컬렉션이나 배열의 중첩 반복을 위한 관용구**

---

```java
for(Face face1 : faces)
	for(Face face2 : faces)
		System.out.println(face1 + " " + face2);
```


<br>

## for-each 문을 사용할 수 없는 경우

1. **파괴적인 필터링(destructive filtering)** - 컬렉션을 순회하면서 선택된 원소를 제거해야 한다면 반복자의 `remove` 메서드를 호출해야 한다. 자바 8 부터는 `Collection`의 `removeIf` 메서드를 사용해 컬렉션을 명시적으로 순회하는 일을 피할 수 있다. 
2. **변형(transforming)** - 리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 한다면 리스트의 반복자나 배열의 인덱스를 사용해야 한다. 
3. **병렬 반복(parallel iteration)** - 여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야 한다.


<br>

### 핵심 정리

> 전통적인 `for` 문과 비교했을 때 `for-each` 문은 명료하고, 유연하고, 버그를 예방해준다. 
성능 저하도 없다. 
가능한 모든 곳에서 `for` 문이 아닌 `for-each` 문을 사용하자
>
