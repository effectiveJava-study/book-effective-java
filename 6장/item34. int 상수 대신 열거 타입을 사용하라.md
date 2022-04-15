# item34. int 상수 대신 열거 타입을 사용하라

열거 타입은 일정 개수의 상수 값을 정의한 다음, <br>
그 외의 값은 허용하지 않는 타임 <br>

ex) 사계절, 태양계의 행성, 카드게임의 카드 종류 등 <br>

자바에서 열거 타입을 지원하기 전에는 <br>
다음 코드처러 정수 상수를 한 묶음 선언해서 사용하곤 했다.

```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 0;
public static final int APPLE_GRANNY_SMITH = 0;


public static final int APPLE_NAVEL = 0;
public static final int APPLE_TEMPLE = 0;
public static final int APPLE_BLOOD = 0;
```

정수 열거 패턴(int enum pattern) 기법의 단점
- 타입 안전을 보장할 방법 없음
- 표현력 좋지 않음
- 동등 연산자(==)로 비교하더라도 경고 메시지 출력하지 않음

정수 대신 문자열 상수를 사용하는 변형 패턴
- 문자열 열거 패턴이라 하는 이 변형은 더 나쁨.
- 상수의 의미를 출력할 수는 있지만
- 오타가 있어도 확인할 길이 없어 자연스럽게 런타임 버그 발생
- 문자열 비교에 따른 성능 저하 역시 당연한 결과

> **열거 타입(enum type)** : 열거 패턴의 단점을 없애고 여러 장점을 안겨주는 대안

```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```
가장 단순한 열거 타입
- 완전한 형태의 클래스
- 다른 언어의 열거 타입보다 훨씬 강력

자바 열거 타입을 뒷받침하는 아이디어
- 열거 타입 자체는 클래스이며
- 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개
- 열거 타입은 밖에서 접근할 수 있는 생성자를 제공하지 않으므로 사실상 final
- 클라이언트가 인스턴스를 직접 생성하거나 확장할 수 없으니 딱 하나씩만 존재

> 싱글턴 : 원소가 하나뿐인 열거 타입 <br>
열거 타입 : 싱글턴을 일반화한 형태

열거 타입의 장점
- 컴파일타임 타입 안전성 제공
- 각자의 이름공간이 있어, 이름이 같은 상수도 평화롭게 공존
(열거 타입에 새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일하지 않아도 됨)
- 열거 타입의 toString 메서드는 출력하기에 적합한 문자열을 내어줌
- 임의의 메서드나 필드 추가 가능
- 임의의 인터페이스 구현 가능
- Object 메스드들을 높은 품질로 구현해 둠
- Comparable(아이템14), Serializable(12장) 구현

---

**어떨 때 열거 타입에 메서드나 필드를 추가하는가?** <br>

Apple과 Orange를 예를 들면, <br>
과일의 색을 알려주거나 과일 이미지를 반환하는 메서드를 추가하고 싶을 수 있다. <br>

또 다른 예로, <br>
태양계의 여덟 행성. <br>
각 행성에는 질량과 반지름이 있고, 이 두 속성을 이용해 표면중력 계산 가능. <br>
어떤 객체의 질량이 주어지면 그 객체가 행성 표면에 있을 때의 무게도 계산 가능

```java
public enum Planet {
	MERCURY(3.302e+23, 2.439e6),
    VENUS(4.869e+24, 6.052e6),
    EARTH(5.975e+24, 6.378e6),
    MARS(6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN(5.685e+26, 6.027e7),
    URANUS(8.683e+25, 2.556e7),
    NEPTUNE(1.024e+26, 2.477e7);
    
    private final double mass; //질량(단위: 킬로그램)
    private final double radius; //반지름(단위: 미터)
    private final double surfaceGravity; //표면중력(단위: m / s^2)
    
    //중력상수(단위:m^3 / kg s^2)
    private static final double G = 6.67300E-11;
    
    //생성자
    Planet(double mass, double radius) {
    	this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }
    
    public double mass()	{ return mass; }
    public double radius()	{ return radius; }
    public double surfaceGravity()	{ return surfaceGravity; }
    
    public double surfaceWeight (double mass) {
    	return mass * surfaceGravity;	// F = ma
    }
}
```

> 열거 타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다.


어떤 객체의 지구에서의 무게를 입력받아 여덟 행성에서의 무게를 출력하는 일의 코드
```java
public class WeightTable {
	public static void main(String[] args) {
    	double earthWeight = Double.parseDouble(args[0]);
        double mass = earthWeight / Planet.EARTH.surfaceGravity();
        for (Planet p : Planet.values())
        	System.out.printf("%s에서의 무게는 %f이다. %n",
            	p, p.surfaceWeight(mass));
    }
}
```

열거 타입은 자신 안에 정의된 상수들의 값을 배열에 담아 반환하는 정적 메서드인 values 제공. <br>
값들은 선언된 순서로 저장.

---
- java.math.RoundingMode & BigDecimal <br>
http://cris.joongbu.ac.kr/course/java/api/java/math/class-use/RoundingMode.html <br>

---

값에 따라 분기하는 열거 타입
```java
public enum Operation {
	PLUS, MINUS, TIMES, DIVIDE;
    
    //상수가 뜻하는 연산 수행
    public double apply(double x, double y) {
    	switch(this) {
        	case PLUS:	return x + y;
            case MINUS :	return x - y;
            case TIMES : 	return x * y;
            case DIVIDE :	return x / y;
        }
       	throw new AssertionError("알 수 없는 연산 : " + this);
    }
}
```
->예쁘지 않고, 런타임 오류 일어날 수 있음 <br>

각 상수에서 자신에 맞게 재정의 하는 방법. <br>

**상수별 메서드 구현**

```java
public enum Operation {
	PLUS {public double apply(double x, double y){return x + y;}},
    MINUS {public double apply(double x, double y){return x - y;}},
    TIMES {public double apply(double x, double y){return x * y;}},
    DIVIDE {public double apply(double x, double y){return x / y;}},

	public abstract double apply(double x, double y);
}
```

상수별 메서드 구현을 상수별 데이터와 결합할 수도 있다. <br>
Operation의 toString을 재정의해 해당 연산을 뜻하는 기호를 반환하도록 하는 예시
```java
public enum Operation {
	PLUS("+") {
    	public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
    	public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
    	public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
    	public double apply(double x, double y) { return x / y; }
    };
    
    private final String symbol;
    
    Operation(String symbol) { this.symbol = symbol; }
    
    @Override public String toString() { return symbol; }
    public abstract double apply(double x, double y);
}
```

toString이 계산식 출력을 얼마나 편하게 해주는지.
```java
public static void main(String[] args) {
	double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    for (Operation op : Operation.values())
    	System.out.printf("%f %s %f = %f%n",
        	x, op, y, op.apply(x, y));
}
```
열거 타입에는 상수 이름을 입력받아 그 이름에 해당하는 상수를 반환해주는 valueOf(String) 메서드가 자동 생성. <br><br>

모든 열거 타입에서 사용할 수 있도록 구현한 fromString 예시. (toString 재정의 방법) - 타입 이름을 적절히 바꿔야 하고 모든 상수의 문자열 표현이 고유해야 함.
```java
private static final Map<String, Operation> stringToEnum = 
	Stream.of(values()).collect(
    	toMap(Object::toString, e->e));
        
//지정한 문자열에 해당하는 Operation을 (존재한다면) 반환한다.
public static Optional<Operation> fromString(String symbol) {
	return Optional.ofNullable(stringToEnum.get(symbol));
}
```

Operation 상수가 stringToEnum 맵에 추가되는 시점 <br>
: 열거 타입 상수 생성 후 정적 필드가 초기화 될 때. <br>

열거 타입의 정적 필드 중 열거 타입의 생성자에서 접근할 수 있는 것은 상수 변수 뿐.(아이템 24)

---

**상수별 메서드 구현에는 열거 타입 상수끼리 코드를 공유하기 어렵다는 단점.**

ex)급여명세서 <br>
열거 타입은 직원의 (시간당) 기본 임금과 그날 일한 시간(분 단위)이 주어지면 일당을 계산해주는 메서드. <br>
주중에 오버타임이 발생하면 잔업수당이 주어지고, <br>
주말에는 무조건 잔업수당이 주어진다.
```java
enum PayrollDay {
	MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;
    
    private static final int MINS_PER_SHIFT = 8 * 60;
    
    int pay(int minutesWorked, int payRate) {
    	int basePay = minutesWorked * payRate;
        
        int overtimePay;
        switch(this) {
        	case SATURDAY : case SUNDAY; //주말
            	overtimePay = basePay / 2;
                break;
            defaul : //주중
            	overtimePay = minutesWorked <= MINS_PER_SHIFT ?
                	0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
        }
        
        return basePay + overtimePay;
    }
}
```
간결하지만, 관리 관점에서는 위험한 코드. <br>
휴가와 같은 새로운 값을 열거 타입에 추가하려면 그 값을 처리하는 case문을 잊지 말고 쌍으로 넣어줘야 하는 것. <br>

상수별 메서드 구현으로 급여를 정확히 계산하는 방법
- 잔업수당을 계산하는 코드를 모든 상수에 중복해서 넣기
- 계산 코드를 평일용과 주말용으로 나눠 각각을 도우미 메서드로 작성한 다음 각 상수가 자신에게 필요한 메서드를 적절히 호출

-> 두 방법 모두 코드가 장황해져 가독성 떨어지고, 오류 발생 가능성 높아짐 <br>

가장 깔끔한 방법 <br>
- 새로운 상수를 추가할 때 잔업수당 '전략'을 선택하도록 하는 것.
잔업수당 계산 -> private 중첩 열거 타입으로 옮기기 (PayType) <br>
PayrollDay 열거 타입의 생성자에서 적당한 것 선택

```java
enum PayrollDay {
	MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY(payType.WEEKEND), SUNDAY(PayType.WEEKEND);
    
    private final PayType payType;
    
    PayrollDay(PayType payType) { this.payType = payType; }
    
    int pay(int minutesWorked, int payRate) {
    	return payType.pay(minutesWorked, payRate);
    }
    
    //전략 열거 타입
    enum PayType {
    	WEEKDAY {
        	int overtimePay(int minsWorked, int payRate) {
            	return minsWorked <= MINS_PER_SHIFT ? 0 :
                	(minsWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
        	int overtimePay(int minsWorked, int payRate) {
            	return minsWorked * payRate / 2;
            }
        };
        
        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;
        
        int pay(int minsWorked, int payRate) {
        	int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }

}
```
---
추가하려는 메서드가 의미상 열거 타입에 속하지 않는다면 직접 만든 열거 타입이라도 이 방식을 적용하는 게 좋다. <br>
> 기존 열거 타입에 상수별 동작을 혼합해 넣을 때는 switch문 선택 <br>
```java
public static Operation invers(Operation op) {
	switch(op) {
    	case PLUS : return Operation.MINUS;
        case MINUS : return Operation.PLUS;
        case TIMES : return Operation.DIVIDE;
        case DIVIDE : return Operation.TIMES;
        default : throw new AssertionError("알 수 없는 연산 : " + op);
    }
}
```

---
그래서 열거 타입 언제 쓰라고? <br>
**필요한 원소를 컴파일타임에 다 알 수 있는 상수 집합이라면, <br>
항상 열거 타입 사용** <br>

태양계 행성, 한 주의 요일, 체스 말처럼 본질적으로 열거 타입인 타입. <br>
메뉴 아이템, 연산 코드, 명령줄 플래그 등 허용하는 값 모두를 컴파일타임에 이미 알고 있을 때도 쓸 수 있음. <br>
<br>
>열거 타입에 정의된 상수 개수가 영원히 고정 불변일 필요는 없다.

---
>**핵심정리** <br>
열거 타입은 확실히 정수 상수보다 뛰어나다. <br>
더 읽기 쉽고 안전하고 강력하다. <br>
대다수 열거 타입이 명시적 생성자나 메서드 없이 쓰이지만, 각 상수를 특정 데이터와 연결짓거나 상수마다 다르게 동작하게 할 때는 필요하다. <br>
드물게는 하나의 메서드가 상수별로 다르게 동작해야 할 때도 있다. <br>
이런 열거 타입에서는 switch문 대신 상수별 메서드 구현을 사용하자. <br>
열거 타입 상수 일부가 같은 동작을 공유한다면 전략 열거 타입 패턴을 사용하자. <br>
