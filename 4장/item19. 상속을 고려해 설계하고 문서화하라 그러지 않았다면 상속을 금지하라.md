#### 클래스를 안전하게 상속할 수 있도록 문서화를 해놓는 것을 권장하고 있습니다. 특히, 재정의 가능 메서드를 호출할 수 있는 모든 상황을 문서로 남겨야함.

#### 즉, 상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지 문서로 남겨야함.

---

**1. 메서드를 재정의하면 어떤 일이 일어나는지 정확히 정리하여 문서로 남겨야 한다. 즉, 상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지 문서로 남겨야 한다.**

* 재정의 가능한 메서드의 API 설명
>* 어떤 순서로 호출하는지
>* 각각의 호출 결과가 이어지는 처리에 어떤 영향을 주는지
>* 재정의 가능 메서드를 호출할 수 있는 모든 상황
>* 메서드의 내부 동작 (Implementation Requirements @implSpec)

---

#### 2. 효율적인 하위 클래스를 어려움 없이 만들 수 있게 하기 위해 클래스의 내부 동작 과정 중간에 끼어들 수 있는 훅(hook)을 잘 선별하여 protected 메서드 형태로 공개해야 할 수도 있다.

>* List의 구현체의 최종 사용자는 removeRange 메서드에 관심이 없다. 그럼에도 이 메서드를 제공하는 이유는 단지 하위 클래스에서 부분리스트의 clear 메서드를 고성능으로 만들기 쉽게 하기 위해서다.
>
>
* removeRange 메서드가 없다면 하위 클래스에서 clear 메서드를 호출하려면 (제거할 원소 수의) 제곱에 비례해 성능이 느려지거나 메커니즘을 밑바닥부터 새로 구현해야 했을 것이다.

---

#### 3. 상속용으로 설계한 클래스는 배포 전에 반드시 하위 클래스를 만들어 검증하라.

>* 꼭 필요한 protected 멤버를 놓쳤다면 하위 클래스를 작성할 때 그 빈자리가 확연히 드러난다. 거꾸로, 하위 클래스를 여러 개 만들 때까지 전혀 쓰이지 않는 protected 멤버는 사실 private이었어야 할 가능성이 크다.

* 이러한 하위 클래스는 3개 이상 작성하되 하나 이상은 제 3자가 작성하는게 적당하다.

* 널리 쓰일 클래스를 상속용으로 설계한다면 반드시 문서화한 내부 사용 패턴과, protected 메서드와 필드를 구현하면서 이 결정이 그 클래스의 성능과 기능에 영원한 족쇄가 될 수 있음을 명시한다.

---

#### 4. 상속용 클래스의 생성자는 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안 된다.

> * 상위 클래스의 생성자가 하위 클래스의 생성자보다 먼저 실행되므로 하위 클래스에서 재정의한 메서드가 하위 클래스의 생성자보다 먼저 호출된다.
>
* 이 때 재정의한 메서드가 하위 클래스의 생성자에서 초기화하는 값에 의존한다면 의도대로 동작하지 않을 것이다.

```
public class Super {
    public Super() {
        overrideMe(); // 생성자가 재정의 가능한 메서드를 호출한다.
    }
    public void overrideMe() {
    }
}
public final class Sub extends Super {
    private final Instacnce instacnce;
    Sub() {
        instacnce = Instacnce.now(); // create instance in constructor
    }
    @Override
    public void overrideMe() {
        System.out.println(instacnce);
    }
    public static void main(String[] args) {
        Sub sub = new Sub();
        /*
         * instance 가 null / instance 두 개 출력 된다
         * 상위 클래스의 생성자는 하위 클래스의 생성자가 인스턴스 필드를 초기화하기 전에 overrideMe 호출
         * */
        sub.overrideMe();
    }
}

```

---

#### 5. clone과 readObject 모두 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안 된다.

>- Clonable이나 Serializable을 구현할지 정해야 한다면 이들을 구현할 때 따르는 제약도 생성자와 비슷하다는 점에 주의해야 한다.
- readObject의 경우 하위 클래스의 상태가 미처 다 역직렬화되기 전에 재정의한 메서드부터 호출하게 된다. 
- clone의 경우 하위 클래스의 clone 메서드가 복제본의 상태를 (올바른 상태로) 수정하기 전에 재정의한 메서드를 호출한다.
- clone이 잘못되어 깊은 복사를 하다가 원본 객체의 일부를 참조하고 있다면 원본 객체에까지도 피해를 줄 수 있다.


```
public class Super implements Cloneable{
    String type;
    public Super() {
        this.type = "super";
    }
    public void overrideMe() {
        System.out.println("super method");
    }
    @Override
    public Super clone() throws CloneNotSupportedException {
        overrideMe();
        return (Super) super.clone();
    }
}
public class Sub extends Super{
  String value;
  @Override
  public void overrideMe() {
    System.out.println("sub mehtod");
    System.out.println(value);  // 테스트 시 이 부분에 null 이 출력 됨
    type = "sub";
  }
  @Override
  public Sub clone() throws CloneNotSupportedException {
    Sub clone = (Sub) super.clone();
    clone.value = "temp";
    return clone;
  }
}
```



---

#### 6. Serializable을 구현한 상속용 클래스가 readResolve나 write Replace 메서드를 갖는다면 이 메서드들은 private이 아닌 protected로 선언해야 한다.

>- private로 선언한다면 하위 클래스에서 무시된다.
>
>
- 상속을 허용하기 위해 내부 구현을 클래스 API로 공개하는 예 중 하나다.




---

>**핵심**
1. 클래스 내부에서 스스로 어떻게 사용하는지 (자기 사용 패턴) 모두 문서로 남겨야 한다.
>
2. 문서화한 것은 그 클래스가 쓰이는 한 반드시 지켜야 한다. (그러지 않으면 그 내부 구현 방식을 믿고 활용하던 하위 클래스를 오동작하게 만들 수 있다.)
>
>
3. 다른 이가 효율 좋은 하위 클래스를 만들 수 있도록 일부 메서드를 protected로 제공해야 할 수도 있다. 
>
>
4. 클래스를 확장해야 할 명확한 이유가 떠오르지 않으면 상속을 금지하자
>
>
5. 상속을 금지하려면 클래스를 final로 선언하.거나 생성자 모두를 외부에서 접근할 수 없도록 만들면 된다.

