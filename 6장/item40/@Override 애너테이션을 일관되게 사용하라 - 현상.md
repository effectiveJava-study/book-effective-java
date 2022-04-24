# 아이템 40
# @Override 애너테이션을 일관되게 사용하라

@Override 애너테이슨을 일관되게 사용하면 여러 가지 악명 높은 버그들을 예방이 가능하다.

```java
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first  = first;
        this.second = second;
    }

    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++)
            for (char ch = 'a'; ch <= 'z'; ch++)
                s.add(new Bigram(ch, ch));
        System.out.println(s.size());
    }
}
```

소문자 2개로 구성된 바이그램 26개를 10번 반복해 집합에 추가한 다음, 그 집합의 크기를 출력하는 프로그램이다. Set은 중복을 허용하지 않으므로 26이 출력될 것 같지만, 실제로는 260이 출력된다.

위 코드에서의 문제점은 equals를 overriding한게 아니라 다중정의(overloading 아이템 52)를 해버려서 문제가 되는것이다.

그래서 equals에 @Override 애너테이션을 달아주면 해결될 것이다.

```java
@Override
    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }
```

위 코드처럼 @Override 애너테이션을 달면 컴파일 오류를 뱉는다.

```java
Bigram.java:10: method does not override or implement a method from a supertype
	@Override public boolean equals (Bigram b) {
  ^
```

위 오류를 해석해보면 “메서드가 슈퍼타입에서 메서드를 재정의하거나 구현하지 않음” 이라고 뜬다.

즉 Bigram 메서드를 받았는데 그게 Bigram 타입인지 확인을 하지 않아서 뜨는 오류인것 같다.

```java
@Override
    public boolean equals(Object o) {
        if (!(o [instanceof](https://arabiannight.tistory.com/313) Bigram))
            return false;
        Bigram b = (Bigram) o;
        return b.first == first && b.second == second;
    }
```

그러니 **상위 클래스의 메서드를 재정의하려는 모든 메서드에  @Override 애너테이션을 달자**

### 핵심 정리

1. 재정의한 모든 메서드에 @Override 애너테이션을 의식적으로 달면 컴파일러가 알려준다.
2. 예외는 구체 클래스에서 상위클래스의 추상 메서드를 재정의한 경우엔 이 애너테이션을 달지  않아도 된다.
