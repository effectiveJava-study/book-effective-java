object
는 객체를 만들 수 있는 구체 클래스.
기본적으로 상속해서 사용하도록 설계되어 있다.

Object 에서 final이 아닌 메서드 (equals, hashCode, toString, clone, finalize) 는 모두 재정의(overriding)를 염두에 두고 설계된 것. -> 재정의 시 지켜야 하는 일반 규약이 명확히 정의되어 있음.




# 아이템 10 
### equals는 일반 규약을 지켜 재정의하라.


>* **equlas? ==?**
기본적으로 양 쪽에 있는 내용을 비교한 값을 boolean type으로 반환한다는 공통점.
- equals는()는 **메소드** : 객체끼리 내용을 비교할 수 있도록 함
- == 은 비교를 위한 **연산자.**
출처 : https://kmj1107.tistory.com/207

equals는 메서드는 재정의하기 쉬워 보이지만 곳곳에 함정이 있다.
다음에 열거한 상황 중 하나에 해당한다면 재정의하지 않는 것이 최선.

1. **각 인스턴스가 본질적으로 고유하다.**
값을 표현하는 게 아니라 동작하는 개체를 표현하는 클래스가 여기 해당.
ex ) Thread는 각각의 Thread가 고유하다.

2. **인스턴스의 '논리적 동치성(logical equality)'을 검사할 일이 없다.**
java.util.regex.Pattern은 equals를 재정의해서 두 Pattern의 인스턴스가 같은 정규표현식을 나타내는지를 검사하고, 즉 논리적 동치성을 검사하는 방법도 있다. 하지만 필요없으면 Object의 기본 equals만으로 해결 가능. 
>?논리적 동치란? -> 'p와 q의 진리값이 서로 같다. p=q라고 표현하기도 하며, p와 q는 같다. 또는 p와 q의 진리값은 같다. 라고 읽는다.' 
출처 : https://bite-sized-learning.tistory.com/363

3. **상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.**
대부분의
Set 구현체는 AbstractSet이 구현한 equals를 상속받아 쓰고,
List 구현체들은 AbstractList로부터, 
Map 구현체들은 AbstracMap으로부터 상속받아 그대로 쓴다.
>AbstractSet : https://runebook.dev/ko/docs/openjdk/java.base/java/util/abstractset
AbstractMap :
https://runebook.dev/ko/docs/openjdk/java.base/java/util/abstractmap
AbstractList :
https://runebook.dev/ko/docs/openjdk/java.base/java/util/abstractlist

4. **클래스가 private 이거나 package-private 이고 equals 메서드를 호출할 일이 없다. **
위험을 철저히 회피하는 스타일이라 equals가 실수로라도 호출되는 걸 막고 싶다면 다음처럼 구현
```
@Oveeride public boolean equals (Object o) {
	throw new AssertionError();	//호출 금지!
}
```

그렇다면 equals를 재정의해야 할 때는 언제인가?
객체 식별성(Object identity : 두 객체가 물리적으로 같은가)이 아니라,
논리적 동치성을 확인해야 하는데,
상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때.

?????
주로 값 클래스가 여기 해당.

값 클래스란 Integer와 String 처럼 값을 표현하는 클래스를 말한다.


equals가 논리적 동치성을 확인하도록 재정의해두면, 그 인스턴스는 값이 같은지를 확인하고 Map의 키와 Set의 원소로 사용할 수 있게 된다.

>**equals 메서드를 재정의할 때 반드시 따라야 하는 일반 규약**
equals 메서드는 동치관계(equivalence relation)를 구현하며, 다음을 만족한다.
- 반사성(reflexivity) : null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true 다.
- 대칭성(symmetry) : null이 아닌 모든 참조 값 x,y에 대해, x.equals(y)가 true면 y.equals(x)도 true다.
- 추이성(transitivity) : null이 아닌 모든 참조 값 x,y,z에 대해, x.equals(y)가 true이고 y.equals(z)도 true면 x.equals(z)도 true다.
- 일관성(consistency) : null이 아닌 모든 참조 값 x,y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
- null-아님 : null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다.

Object 명세에서 말하는 동치관계란?
집합을 서로 같은 원소들로 이뤄진 부분집합으로 나누는 연산.
이 부분집합을 동치류(equivalence class; 동치 클래스)라 한다.

equals 메서드가 쓸모 있으려면 모든 원소가 같은 동치류에 속한 어떤 원소와도 서로 교환할 수 있어야 한다.

- 반사성 : 객체는 자기 자신과 같아야 한다.
- 대칭성 : 두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다는 뜻.
equals 는 대소문자를 무시하기 때문에 자칫 어길 수 있어 보임.
( ex : 잘못된 코드 - 대칭성 위배 )
```
public final class CaseInsensitiveString {
	private final String s;
    
    public CaseInsensitiveString(String s) {
    	this.s = Objects.requireNonNull(s);
    }
    
    // 대칭성 위배!
    @Override public boolean equals(Object o) {
    	if (o instanceof CaseInsensitiveString)
        	return s.equalsIgnoreCase(
            	((CaseInsensitiveString) o).s);
        if (o instanceof String) // 한 방향으로만 작동한다!
        	return s.equalsIgnoreCase((String) o);
       	return false;
    }
    .... // 나머지 코드는 생략
}
```
requireNonNull(s) : 
https://multifrontgarden.tistory.com/205
CaseInsensitiveString : 
https://www.javadoc.io/doc/com.cedarsoftware/java-util/1.15.0/com/cedarsoftware/util/CaseInsensitiveMap.CaseInsensitiveString.html

CaseInsensitiveString의 equals는 순진하게 일반 문자열과도 비교를 시도.

CaseInsensitiveString cis = new CaseInsenstiveString("Polish");
String s = "polish";


예상할 수 있듯 cis.equals(s)는 true를 반환한다.

그러나
CaseInsenstiveString의 equals는 일반 String을 알고 있지만 (polish를 알고 있지만)
String의 equals는 CaseInsenstivieString의 존재를 모른다. (Polish를 모름)

따라서 s.equlas(cis)는 false를 반환하여, 대칭성을 명백히 위반한다.

--> 문제 해결
```
@Override public boolean equals(Object o) {
	return o instanceof CaseInsensitiveString &&
    	((CaseInsensitiveString) o).s.equlasIgnoreCase(s);
}
```

- 추이성 : 첫 번째 객체와 두 번째 객체가 같고, 두 번째 객체와 세 번째 객체가 같다면, 첫 번째 객체와 세 번째 객체도 같아야 한다는 뜻.

( ex : 2차원에서의 점을 표현하는 클래스; equals 비교에 영향을 주는 정보 추가하는 예시 )
```
public class Point {
	private final int x;
    private final int y;
    
    public Point(int x, int y) {
    	this.x = x;
        this.y = y;
    }
    
    @Override public boolean equals(Object o) {
    	if (!(o instanceof Point))
        	return false;
       	Point p = (Point)o;
        return p.x == x && p.y == y;
    }
    ... // 나머지 코드는 생략
}
```

점에 색상을 더해보자.
```
public class ColorPoint extends Point {
	private final Color color;
    
    public ColorPoint(int x, int y, Color color) {
    	super(x, y);
        this.color = color;
    }
    ... // 나머지 코드는 생략
}
```

그대로 두면 Point의 구현이 상속되어 색상 정보는 무시한 채 비교를 수행한다. equals 규약을 어긴 것은 아니지만, 중요한 정보를 놓치게 되니 받아들일 수 없는 상황.

다음 코드처럼 비교 대상이 또 다른 ColorPoint이고 위치와 색상이 같을 때만 true를 반환하는 equals를 생각해보자.

( ex : 잘못된 코드 - 대칭성 위배! )
```
@Override public boolean equals(Object o) {
	if (!(o instanceof ColorPoint))
    	return false;
   	return super.equals(o) && (ColorPoint) o).color == color;
}
```

이 메서드는 일반 Point를 ColorPoint에 비교한 결과와 그 둘을 바꿔 비교한 결과가 다를 수 있다.
Point의 equals는 색상을 무시하고, ColorPoint의 equals는 입력 매개변수의 클래스 종류가 다르다며 매번 false만 반환할 것.

각각의 인스턴스를 하나씩 만들어 실제로 동작하는 모습 확인하는 방법

Point p = new Point(1, 2);
ColorPoint cp = new ColorPoint(1, 2, Color.RED);

이제 p.equals(cp)는 true를,
cp.equals(p)는 false를 반환한다.
ColorPoint.equals가 Point와 비교할 때는 색상을 무시하도록 하면 해결되나?

( ex : 잘못된 코드 - 추이성 위배 )
```
@Override public boolean equals(Object o) {
	if (!(o instanceof Point))
    	return false;
        
   	// o가 일반 Point면 색상을 무시하고 비교한다.
    if (!(o instanceof ColorPoint))
    	return o.equals(this);
        
  	// o가 ColorPoint면 색상까지 비교한다.
    return super.equals(o) && ((ColorPoint) o).color == color;
}
```

이 방식은 대칭성은 지켜주지만, 추이성을 깬다.

ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
Point p2 = new Point(1, 2);
ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);

이제 p1.equals(p2)와 p2.equals(p3)는 true를 반환하는데,
p1.equals(p3)가 false를 반환한다.
추이성에 명백히 위배.

p1과 p2, p2와 p3 비교에서는 색상을 무시했지만, p1과 p3 비교에서는 색상까지 고려했기 때문.

또한, 이 방식은 무한 재귀에 빠질 위험도 있다.

사실 이 현상은 모든 객체 지향 언어의 동치관계에서 나타나는 근본적인 문ㄴ제이다.
>**구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.**

값을 추가하지 않는 방식으로 Point를 확장해보자.

```
public class CounterPoint extends Point {
	private static final AtomicInteger counter = new AtomicInteger();
    
    public CounterPoint(int x, int y) {
    	super(x, y);
        counter.incrementAndGet();
    }
    public static int numberCreated() {return counter.get();}
}
```

리스코프 치환 원칙(Liskov substitution principle)에 따르면, 어떤 타입에 있어 중요한 속성이라면 그 하위 타입에서도 마찬가지로 중요하다.
따라서 그 타입의 모든 메서드가 하위 타입에서도 똑같이 잘 작동해야 한다.

구체 클래스의 하위 클래스에서 값을 추가할 방법은 없지만 괜찮은 우회 방법이 하나 있다.
"상속 대신 컴포지션을 사용하라(아이템 18)"의 조언.

Point를 상속하는 대신 Point를 ColorPoint의 private 필드로 두고, ColorPoint와 같은 위치의 일반 Point를 반환하는 뷰(view) 메서드를 public으로 추가하는 식

( ex : equals 규약을 지키면서 값 추가하기 )
```
public class ColorPoint {
	private final Point point;
    private final Color color;
    
    public ColorPoint (int x, int y, Color color) {
    	point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }
    
    /**
    *	이 ColorPoint의 Point 뷰를 반환한다.
    */
    public Point asPoint() {
    	return point;
    }
    
    @Override public boolean equals(Object o) {
    	if (!(o instanceof ColorPoint))
        	return false;
      	ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.color.equals(color);
    }
    ... // 나머지 코드는 생략
}
```

- 일관성 : 두 객체가 같다면 (어느 하나 혹은 두 객체 모두가 수정되지 않는 한) 앞으로도 영원히 같아야 한다는 뜻

가변 객체는 비교 시점에 따라 서로 다를 수도 혹은 같을 수도 있는 반면, 불변 객체는 한번 다르면 끝까지 달라야 한다.

클래스가 불변이든 가변이든 equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안 된다. 이 제약을 어기면 일관성 조건을 만족시키기가 어려움.

java.net.URL의 equals는 주어진 URL과 매핑된 호스트의 IP 주소를 이용해 비교한다.
호스트 이름을 주어진 URL과 매핑된 호스트의 IP 주소를 이용해 비교한다.
허나 따라하지 말 것.

- Null-아님 : 이름처럼 모든 객체가 null과 같지 않아야 한다는 뜻.
수많은 클래스가 다음 코드처럼 입력이 null인지를 확인해 자신을 보호한다.
```
// 명시적 null 검사 - 필요 없다!
@Override public boolean equals(Object o) {
	if (o == null)
    	return false;
        ...
}
```
이러한 검사보다는 동치성을 검사하려면
equals는 건네받은 객체를 적절히 형변환 후 필수 필드들의 값을 알아내야 한다.

그러려면 형변환에 앞서 instanceof 연산자로 매개변수가 올바른 타입인지 검사해야 한다.

```
// 묵시적 null 검사 - 이쪽이 낫다.
@Override public boolean equals(Object o) {
	if (!(o instanceof MyType))
    	return false;
 	MyType mt = (MyType) o;
    ...
}
```

>**equals 메서드 구현 방법을 단계별로 정리**
1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
자기 자신이면 true를 반환한다.
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
그렇지 않다면 false를 반환한다.
3. 입력을 올바른 타입으로 형변환한다.
2번에서 instanceof 검사를 했기 때문에 이 단계는 100% 성공한다.
4. 입력 객체와 자기 자신의 대응되는 '핵심'필드들이 모두 일치하는지 하나씩 검사한다.
모든 필드가 일치하면 true, 하나라도 다르면 false를 반환한다.

float와 double을 제외한 기본 타입 필드는 == 연산자로 비교하고, 
참조 타입 필드는 각각의 equals 메서드로, 
float와 double 필드는 각각 정적 메서드인 Float.compare(float, float)와 Double.compare(double, double)로 비교한다.

특수한 부동소수 값 등을 다뤄야 하기 때문.

배열 필드는 원소 각각을 앞서의 치짐대로 비교한다.
배열의 모든 원소가 핵심필드라면 Arrays.equals 메서드들 중 하나를 사용.

때론 null도 정상 값으로 취급하는 참조 타입 필드도 있다.
이런 필드는 정적 메서드인 Object.equals(Object, Object)로 비교해 NPE 발생 예방.

CaseInsensitiveString 예처럼 비교하기 복잡한 필드를 가진 클래스라면, 그 필드의 표준형(canonical form)을 저장해둔 후 표준형끼리 비교하면 훨씬 경제적.
> 표준형
: 하나만 존재해야 하고, 객체에서 표준형을 사용하면, 반드시 약속한 대로(예상한 대로) 동작해야 한다.
출처 : https://github-wiki-see.page/m/java-squid/effective-java/wiki/2020.10.17

가변 객체라면 값이 바뀔 때마다 표준형을 최신 상태로 갱신해줘야 한다.
> 가변 객체 : 값을 수정할 수 있는 객체 (list, set, dic)
불변 객체 : 값을 수정할 수 없는 객체 (int, float, bool, tuple ..)

equals를 다 구현했다면 세 가지 질문.
대칭적인가? 추이성이 있는가? 일관적인가?

세 요건 중 하나라도 실패한다면 원인 찾아서 고치기.

( ex : 전형적인 equals 메서드의 예 )
```
public final class PhoneNumber {
	private final short areaCode, prefix, lineNum;
    
    public PhoneNumber(int areaCode, int prefix, int lineNum) {
    	this.areaCode = rangeCheck(areaCode, 999, "지역코드");
        this.prefix = rangeCheck(prefix, 999, "프리픽스");
        this.lineNum = rangeCheck(lineNum, 9999, "가입자 번호");
    }
    
    private static short rangeCheck(int val, int max, String arg) {
    	if (val < 0 || val > max)
        	throw new IllegalArgument Exception(arg + ": " + val);
            return (short) val;
    }
    
    @Override public boolean equals(Object o) {
    	if (o == this)
        	return true;
      	if (!(o instanceof PhoneNumber))
        	return false;
      	PhoneNumber pn = (PhoneNumber)o;
        return pn.lineNum == lineNum && pn.prefix == prefix 
        	&& pn. areaCode == areaCode;
    }
    ... // 나머지 코드는 생략
}
```

IllegalArgumentException : 
https://help.acmicpc.net/judge/rte/IllegalArgumentException


마지막 주의 사항.
- equals를 재정의할 땐 hashCode도 반드시 재정의하자.
- 너무 복잡하게 해결하려 들지 말자.
필드들의 동치성만 검사해도 equals 규약을 어렵지 않게 지킬 수 있다.
- Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자.
```
// 잘못된 예 - 입력 타입은 반드시 Object여야 한다!
public boolean equals(MyClass o) {
	...
}
```


>**핵심정리**
꼭 필요한 경우가 아니면 equals를 재정의하지 말자.


