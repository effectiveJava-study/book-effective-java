## 40  @Override 애너테이션을 일관되게 사용하라.

@Override는 메서드 선언에만 달 수 있으며 상위 타입 메서드를 재정의했음을 의미한다.
이를 일관되게 사용해야 버그를 예방할 수 있다.



```
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first = first;
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
        for (int i = 0; i < 10; i++) {
            for (char ch = 'a'; ch <= 'z'; ch++) {
                s.add(new Bigram(ch, ch));
            }
        }

        System.out.println(s.size());
    }
}
```
이 코드는 a-z까지 소문자 26개를 10번 반복해 집합에 추가한다음 크기를 출력하는데 Set이 중복을 허용하지 않으니 26이 출력될 것 같을것이다.

하지만 결과는 260이 나온다.

잘 보면 equals에 문제가 있다.
1. @Override를 적어주지 않았다.
2. Object의 equals를 재정의하기 위해 Object를 매개변수로 받아줘야 하는데 Bigram을 받아줬다.
3. 위 과정을 마친 후 Object에 대한 타입 체크를 선행해야 한다.

다시 고쳐보면 다음과 같다.

```
@Override
public boolean equals(Object o) {
    if (!(o instanceof Bigram))
        return false;
    Bigram b =  (Bigram) o;
    return b.first == first && b.second == second;
}
```

구체 클래스에서 상위 클래스의 추상 메서드를 재정의할 때를 제외하고는 상위 클래스의 메서드를 재정의하려는 모든 메서드에 @Override 애너테이션을 달자. 어차피 구체클래스에서 구현하지 않았으면 IDE가 알려준다.

추상 클래스나 인터페이스에서는 상위 클래스나 상위 인터페이스의 메서드를 재정의하는 모든 메서드에 @Override를 적어주는 것이 좋은데 실수로 추가한 메서드가 없음을 보장하기 위해서다.


`정리`
- 재정의한 모든 메서드에 @Override를 달면 컴파일러가 실수를 알려준다.
- 구체 클래스에서 상위 클래스의 추상 메서드를 재정의한 경우는 달지 않아도 된다.
