# item 35.ordinal 메서드 대신 인스턴스 필드를 사용하라.

열거 타입 상수는 자연스럽게 하나의 정숫값에 대응.

모든 열거 타입은 해당 상수가 그 열거 타입에서 몇 번째 위치인지 반환하는 ordinal이라는 메서드 제공.

~~~java
public  enum Ensemble {
 SOLO, DUET, TRIO, QUARTET, QUINET,
 SEXTET, SEPTET, OCTET, NONET, DECTET;

 public int numberOfNusicians() {.return ordinal() + 1; }
}
~~~

동작은 하지만 유지보수하기가 끔찍한 코드.

상수 선언 순서를 바꾸는 순간, 오동작

값을 중간에 비워둘 수 없음

—> 코드가 깔끔하지 못할 뿐아니라, 쓰이지 않는 값이 많아질수록 실용성 떨어짐.

---

열거 타입 상수에 연결된 값은 ordinal 메서드로 얻지 말고, 인스턴스 필드에 저장.

~~~java
public enum Ensemble {
 SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
 SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
 NONET(9), DECTET(10), TRIPLE_QUARTET(12);

 private final int numberOfMusicians;
 Ensemble(int size) {.this.numberOfMusicians = size; }
 public int numberOfMusicians() {.return numberOfMusicians; }
}
~~~

---

EnumSet EnumMap 같이 열거 타입 기반의 범용 자료구조에 쓸 목적으로 설계되었기 때문에, 이런 용도가 아니라면 ordinal 메서드는 절대 사용하지 말자.

---

Ordinal 메서드
- https://altongmon.tistory.com/113
