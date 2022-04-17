# 6장 열거 타입과 애너테이션

자바에는 특수한 목적의 참조 타입이 두 가지 있는데 하나는 열거타입(enum), 두 번쨰는 애너테이션이다.
이를 올바르게 사용하는 방법에 대해 알아보자.

## 34 상수 대신 열거 타입을 사용하라.
열거 타입은 enum이다. 일정한 상수 값을 정의한 다음 그 외의 값은 허용하지 않는 타입이다.
자바의 열거 타입 지원 전에는 정수 상수를 한 묶음으로 처리했었다.
```
    public static final int APPLE_FUJI = 0;
    public static final int APPLE_PIPPIN = 1;
    public static final int APPLE_GRANNY_SMITH = 2;

    public static final int ORANGE_NAVEL = 0;
    public static final int ORANGE_TEMPLE = 0;
    public static final int ORANGE_BLOOD = 0;

```

이는 타입 안전을 보장할 수 없으며 

```
    public void apples(int apple_type){
    }
    apples(ORANGE_NAVEL)
```
-> 아무 에러가 나지 않는다.

표현성도 좋지 않다. 
```
    @Test
    void printTest() {
        System.out.println(enums.APPLE_FUJI);
        System.out.println(enums.Apple.FUJI);
    }
```
위의 출력 결과는 다음과 같다.

`0`<br>
`FUJI`

추가로 다른 타입과 동등연산자 비교에도 컴파일 오류가 나지 않는다.
따라서 이를 보완해서 나온 것이 열거 타입이다.

```
    public enum Apple{FUJI, PIPPIN, GRANNY_SMITH}
    public enum Orange{NAVEL, TEMPLE, BLOOD}
```

열거 타입 자체는 클래스이며 상수 하나당 인스턴스로 만들어 public static final인 싱글턴이라고 생각하면 된다.

인자 태양계 행성으로 열거 타입을 만들어보자
```
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
위의 코드는 지구에서의 무개를 입력받아 여덟 행성에서의 무게를 출력하는 코드로 생성자에서 표면중력을 계산해서 저장해두고 각 행성 
표면에서의 무게를 필요로 할 때 그를 이용해 계산하는 타입이다.

```
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
열거타입은 자신 안에 정의된 상수를 배열에 담아 반환하는 정적 메서드 values를 제공한다.
toString은 상수 이름을 문자열로 반환해 println등을 사용하기에 좋다.

위의 코드가 명왕성을 삭제한 코드라고 생각해보자. 명왕성을 삭제할 때 어떤 일이 일어났을까?
제거된 상수를 참조하는 클라이언트에서는 컴파일 오류가 발생한다. 

만약 상수마다 동작이 달라지기를 바란다면 그것도 가능하다.
```
public enum Operation {
    PLUS, MINUS, TIMES, DIVIDE;

    public double apply(double x, double y){
        switch (this){
            case PLUS: return x + y;
            case MINUS: return x - y;
            case TIMES: return x * y;
            case DIVIDE: return x / y;
        }
        throw new AssertionError("알 수 없는 연산" + this);
    }
}

```
물론 이건 좋지 않은 구현이다. 새로운 상수 추가시 case문을 추가해야되며 빼먹는다면 런타임 오류를 낸다.
다행히 더 나은 방법이 있다. 열거 타입에 apply라는 추상 메서드를 선언하고 상수에서 재정의하는 것이다.

```
public enum AdvancedOperation {
    PLUS {public double apply(double x, double y){return x + y;}},
    MINUS {public double apply(double x, double y){return x - y;}},
    TIMES {public double apply(double x, double y){return x * y;}},
    DIVIDE {public double apply(double x, double y){return x / y;}};

    public abstract double apply(double x, double y);

}
```
만약 재정의하지 않는다면 컴파일 오류가 나기 때문에 더 낫다.

```
public enum AdvancedOperation {
    PLUS ("+") {public double apply(double x, double y){return x + y;}},
    MINUS ("-") {public double apply(double x, double y){return x - y;}},
    TIMES ("*") {public double apply(double x, double y){return x * y;}},
    DIVIDE ("/") {public double apply(double x, double y){return x / y;}};

    private final String symbol;

    AdvancedOperation(String symbol){this.symbol = symbol;}

    @Override
    public String toString() {return symbol;}
    public abstract double apply(double x, double y);
}
```


```
private static final Map<String, Operation> stringToEnum =
	Stream.of(values()).collect(Collectors.toMap(Object::toString, e -> e));
```
풀면 아래와 같다.
```
private static final Map<String, Operation> stringToEnum =
    Stream.of(values()).collect(Collectors.toMap(new Function<Operation, String>() {
        @Override
        public String apply(Operation o) {
            return o.toString();
        }
    }, new Function<Operation, Operation>() {
        @Override
        public Operation apply(Operation o) {
            return o;
        }
    }));
```

값에 따라 분기하며 코드를 공유하는 열거 타입
```
public enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;

    private static final int MINS_PER_SHIFT = 8 * 60;

    int pay(int minutesWorked, int payRate) {
        int basePay = minutesWorked * payRate;

        int overtimePay;
        switch(this) {
            case SATURDAY : case SUNDAY: //주말
                overtimePay = basePay / 2;
                break;
            default : //주중
                overtimePay = minutesWorked <= MINS_PER_SHIFT ?
                    0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
        }
        return basePay + overtimePay;
    }
}
```

```
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


```
enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY,
    SATURDAY(PayType.WEEKEND), SUNDAY(PayType.WEEKEND);

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

```
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
