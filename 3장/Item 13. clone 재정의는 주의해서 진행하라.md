### **아이템 13**

# **clone 재정의는 주의해서 진행하라**

### Clone()

- 자신을 복제하여 새로운 인스턴스를 생성하는 일을 한다.
- **clone**은 Object에 정의 되어 있고 쓰고싶으면 **Cloneable**을 구현해야만 한다!

**Cloneable**은 복제해도 되는 클래스임을 명시하는 용도의 **믹스인 인터페이스
Object에 정의 되어있는 clone을 사용하기 위한 인터페이스**

→ 의도한 목적을 제대로 이루지 못함!

### 믹스인 인터페이스란

- 특정 행동을 실행해주는 메소드를 제공하는데 단독으로 쓰이지 않고, 다른 클래스에 행동을 더해주는 용도로 사용
- 특정 기능만 담당하는 클래스, 단독 사용이 아닌 다른 클래스에 사용될 목적으로 작성된 부품 같은 클래스를 의미

<br><br>

**가장 큰 문제는 clone 메서드가 선언된 곳이 Cloneable이 아닌 Object이고, 그마저도 protected이다.** 

→ Cloneable을 구현하는 것만으로는 외부 객체에서 clone 메서드를 호출할 수 없다. 

→ 리플렉션을 사용하면 가능하지만, 100% 성공하는 것이 아님.

### Clonable 인터페이스

- 메서드 하나 없은 Cloneable 인터페이스는 Object의 protected 메서드인 clone의 동작방식을 결정한다.
- Cloneable을 구현한 클래스의 인스턴스에서 clone을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환하며, 그렇지 않은 클래스의 인스턴스를 호출하면 CloneNotSupportedException을 던진다.

→ 인터페이스를 구현한다는 것은 일반적으로 해당 클래스가 그 인터페이스에서 정의한 기능을 제공한다고 선언하는 행위인데, Cloneable같은 경우에는 상위 클래스에 정의된 protected 메서드의 동작 방식을 변경했다 = 따라하지 말자 

실무에서는 Cloneable을 구현한 클래스는 clone 메서드를 public으로 제공하며, 사용자는 당연히 복제가 제대로 이뤄지리라 기대한다. 

→ 그 클래스와 모든 상위 클래스는 복잡하고, 강제할 수 없고, 허술하게 기술된 프로토콜을 지켜야만한다. 
→ 깨지기 쉽고, 위험하고, 모순적인 매커니즘 탄생
→ 생성자를 호출하지 않고도 객체를 생성할 수 있게 된다. 

<br>

### clone메서드의 일반 규약은 허술
Object 명세에서 가져온 설명들

```java
x.clone() != x                       //참
복사한 객체와 원본 객체는 서로 다른 객체이다.

x.clone().getClass == x.getClass()   //참
반드시 만족해야 하는 것은 아니다. 

x.clone().equals(x)                  //참
복사한 객체와 원본객체는 논리적 동치성이 같다. 

x.clone().getClass() == x.getClass() //참
관례상, 이 방법으로 반환된 객체는 독립성이 있어야 한다. 
이를 만족하려면 super.clone으로 얻는 객체의 필드 중 하나 이상을 반환전에 수정해야 할 수도 있다. 
```

- 강제성이 없다는 점만 빼면 생성자 연쇄(constructor chaining)와 비슷한 메커니즘
- 강제성이 없어서 clone 메서드가 super.clone이 아닌, 생성자를 호출해 얻은 인스턴스를 반환해도 컴파일러는 문제가 없다고 판단한다.
- 어떤 클래스의 하위 클래스에서 super.clone을 호출한다면 잘못된 클래스의 객체가 만들어져, 결국 하위 클래스의 clone 메서드가 제대로 동작하지 않게 된다는 점을 주의해야한다.

<br>

### 가변 상태를 참조하지 않는 클래스용 clone 메서드

```java
public class PhoneNumber implements Cloneable {
    @Override
    protected PhoneNumber clone() {
        try {
            return (PhoneNumber) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError(); // 일어날 수 없는 일이다.
        }
    }
}
```

- 모든 필드가 기본 타입이거나 불변 객체를 참조한다면 이 객체는 완벽히 우리가 원하는 상태라 더 손볼 것이 없다.
- 쓸데없는 복사를 지양한다는 관점에서 보면 불변 클래스는 굳이 clone메서드를 제공하지 않는게 좋다.
- 이 메서드가 동작하게 하려면 PhoneNumnber의 클래스 선언에 Cloneable 구현을 추가해야 한다.
- 재정의한 메서드의 반환 타입은 상위 클래스의 메서드가 반환하는 타입의 하위 타입일 수 있다. 
이 방식으로 클라이언트가 형변환하지 않아도 되게끔 한다.

<br>

### Stack 클래스

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CPAPCITY];
    }

    public Object push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }

    //원소를 위한 공간을 적어도 하나 이상 확보한다. 
    private void ensureCapacity() {
        if (elements.length == size)
        elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

**이 클래스를 복제할 수 있도록 해보자** 

- 반환된 Stack 인스턴스의 size필드는 올바른 값을 갖겠지만, elements 필드는 원본 Stack 인스턴스와 똑같은 배열을 참조할 것이다.
- 원본이나 복제본 중 하나를 수정하면 다른 하나도 수정되어 불변식을 해친다는 이야기이다.
- 따라서 프로그램이 이상하게 동작하거나 NullPointerException을 던질 것이다.

### 가변 상태를 참조하는 클래스용 clone 메서드

- clone 메서드는 사실상 생성자와 같은 효과를 낸다.
- 즉, clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다.

Stack의 clone 메서드는 제대로 동작하려면 **스택 내부 정보를 복사**해야만 한다. 

이때 가장 쉬운 방법은 **elements 배열의 clone을 재귀적으로 호출해주는 것**이다. 

```java
public class Stack implements Cloneable {
    private Object[] elements;

    @Override
    protected Stack clone() {
        try {
            Stack result = (Stack) super.clone();
            result.elements = elements.clone();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

- 배열의 clone은 런타임 타입과 컴파일타입 타입 모두가 원본 배열과 똑같은 배열을 반환한다. 
→ 따라서 배열을 복제할 때는 배열의 clone메서드를 사용하라고 권장!
- **배열은 clone 기능을 제대로 사용하는 유일한 예!**

<br>

### elements 필드가 final일 경우

- final 필드에는 새로운 값을 할당할 수 없기 때문이다.
- 직렬화와 마찬가지로 Cloneable 아키텍처는 ‘가변 객체를 참조하는 필드는 final로 선언하라’는 일반 용법과 충돌한다.
- 그래서 복제할 수 있는 클래스를 만들기 위해 일부 필드에서 final 한정자를 제거해야 할 수도 있다.

<br>

### 해시 테이블용 clone 메서드

- 해시 테이블 내부는 버킷들의 배열이고, 각 버킷은 키-값 쌍을 담는 연결리스트의 첫 번째 엔트리를 참조한다.
- 복제본은 자신만의 버킷 배열을 갖지만, 이 배열은 원본과 같은 연결 리스트를 참조하여 원본과 복제본 모두 예기치 않게 동작할 가능성이 생긴다.

**private 클래스인 HashTable.Entry는 깊은 복사(deep copy)를 지원하도록 보강 되었다.** 

- Entry의 deepCopy 메서드는 자신이 가리키는 연결리스트 전체를 복사하기 위해 자신을 재귀적으로 호출한다. 
→ 버킷이 너무 길지 않다면 잘 작동 
→ 재귀 호출 때문에 리스트의 원소 수만큼 스택 프레임을 소비하여, 리스트가 길면 스택 오버플로를 일으킬 위험이 있기 때문
- 이 문제를 피하려면 deepCopy를 재귀 호출 대신 반복자를 써서 순회하는 방향으로 수정해야 한다.

<br>

### 복잡한 가변 객체를 복제하는 마지막 방법

- super-clone을 호출하여 얻은 객체의 모든 필드를 초기 상태로 설정한 다음, 원본 객체의 상태를 다시 생성하는 고수준 메서드들을 호출한다. 
ex) HashTable에서라면, buckets 필드를 새로운 버킷배열로 초기화 후, put 메서드 호출
- 고수준 API를 활용해 복제를 하면 보통은 간단하고 제법 우아한 코드를 얻게 되지만, 느리다.

---

### 생성자에서는 재정의 될 수 있는 메서드를 호출하지 않아야 하는데 clone 메서드도 마찬가지이다.

- clone이 하위 클래스에서 재정의한 메서드를 호출하면, 하위 클래스는 복제 과정에서 자신의 상태를 교정할 기회를 잃게 되어 원본과 복제본의 상태가 달라질 가능성이 크다.
- 따라서 메서드를 호출하더라도 해당 메서드는 재정의할 수 없는 **final**이거나 **private**메서드여야 한다.
- Object의 clone 메서드는 CloneNotSupportedException을 던진다고 선언했지만, 재정의한 메서드는 그렇지 않다. 
public인 clone 메서드에서는 throws절을 없애야 한다. 
→ 검사 예외를 던지지 않아야 그 메서드를 사용하기 편하기 때문

<br>

### 상속해서 쓰기 위한 클래스 설계 방식 두 가지 중 어느 쪽에서든, 상속용 클래스는 Cloneable을 구현해서는 안된다.

- Object방식을 모방하여, 제대로 작동하는 clone메서드를 구현해 protected로 두고 CloneNotSuppoertedException도 던질 수 있다고 선언한다. 
→ Object를 바로 상속할 때 처럼 Cloneable 구현 여부를 하위 클래스에서 선택하도록 해준다.
- clone을 동작하지 않게 구현해놓고 하위클래스에서 재정의 하지 못하게 할수도 있다.

<br>

**하위 클래스에서Cloneable을 지원하지 못하게 하는 clone 메서드**

```java
public class CloneNotSupportedExample implements Cloneable {
    @Override
    protected final Object clone() throws CloneNotSupportedException {
        throw new CloneNotSupportedException();
    }
}
```

<br>

### **Cloneable을 구현한 스레드 안전 클래스를 작성할 때는 clone메서드 역시 적절한 동기화 해줘야 한다.**

- Object의 clone메서드는 동기화를 신경 쓰지 않았다.
- super.clone호출 외에 다른 할 일이 없더라도 clone을 재정의하고 동기화 해줘야한다.

<br>

### Cloneable을 구현하는 모든 클래스는 clone을 재정의 해야 한다!

- 접근 제한자는 public, 반환 타입은 클래스 자신으로 변경 필요
- 이 메서드는 가장 먼저 super.clone을 호출한 후 필요한 필드를 전부 적절히 수정
- 그 객체의 내부 ‘깊은 구조’에 숨서있는 모든 가변객체를 복사하고, 복제본이 가진 객체 참조 모두가 복사된 객체들을 가리키게 함을 뜻한다.
- 이러한 내부 복사는 주로 clone을 재귀적으로 호출해 구현하지만, 최선은 아니다
- 기본 타입 필드와 불변 객체 참조만 갖는 클래스라면 아무 필드도 수정할 필요가 없다. 
(일변 번호나 고유 ID는 비록 기본 타입이나 불변일지라도 수정 필요)

---

### **복사 생성자와 복사 팩터리**

- Cloneable을 이미 구현한 클래스를 확장한다면 어쩔 수 없이 clone을 잘 작동하도록 구현해야 한다.
- 그렇지 않은 상황에서는 복사 생성자와 복사 팩터리라는 더 나은 객체 복사 방식을 제공할 수 있다.

**복사 생성자**란 단순히 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자를 말한다. 

```java
public class Yum {

    public Yum(Yum yum) {
        // ...
    }
}
```

**복사 팩터리**란 복사 생성자를 모방한 정적 패터리다. 

```
public class Yum {
    public static Yum newInstance(Yum yum) {
        // ...
    }
}
```

### **복사 생성자, 복사 팩터리 VS Clonable/clone**

복사 생성자와 그 변형인 복사 팩터리는 Cloneable/clone 방식보다 나은면이 많다. 

- 언어 모순적이고 위험천만한 객체 생성 메커니즘(생성자를 쓰지 않는 방식)을 사용하지 않는다.
- 엉성하게 문서화된 규약에 기대지 않는다.
- 정상적인final 필드 용법과도 충돌하지 않는다.
- 불필요한 검사 예외를 던지지 않고, 형변환도 필요치 않다.
- 해당 클래스가 구현한 ‘인터페이스’ 타입의 인스턴스를 인수로 받을 수 있다.

## 핵심 정리

- 새로운 인터페이스를 만들 때는 절대 Cloneable을 확장해서는 안된다.
- 새로운 클래스도 이를 구현해서는 안된다.
- final 클래스라면 Cloneable을 구현해도 위험이 크지 않지만, 성능 최적화 관점에서 검토한 후 별다른 문제가 없을 때만 드물게 허용해야 한다.
- 기본 원칙은 **‘ 복제 기능은 생성자와 팩터리를 이용하는 게 최고’** 라는 것이다.
- 배열만은 clone메서드 방식이 가장 깔끔한, 이 규칙의 합당한 예외라 할 수 있다.
