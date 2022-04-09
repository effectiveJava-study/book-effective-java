# 아이템 23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라

```java
class Figure {
	enum Shape { RECTANGLE, CIRCLE };
    
    //태그 필드 - 현재 모양을 나타낸다.
    final Shape shape;
    
	//다음 필드들은 모양이 사각형(RECTANGEL)일 때만 쓰인다.
    double lenght;
    double width;
    
    //다음 필드는 모양이 원(CIRCLE)일 때만 쓰인다.
    double radius;
    
    //원용 생성자
    Figure(double length, double width) {
    	shape = Shape.RECTANGE;
        this.length = length;
        this.width = width;
    }
    
    double area() {
    	switch(shape) {
        	case RECTANGEL:
            	return length * width;
           	case CIRCLE:
            	return Math.PI * (radius*radisu);
          	default:
            	throw new AssertionError(shape);
        }
    }
}
```

필드들을 final로 선언하려면 해당 의미에 쓰이지 않는 필드들까지 생성자에서 초기화 해야 한다. <br>
(쓰지 않는 필드를 초기화하는 불필요한 코드가 늘어난다.)

예를 들어, <br>
새로운 의미를 추가할 때마다 모든 switch 문을 찾아 <br>
새 의미를 처리하는 코드를 추가해야 하는데, <br>
하나라도 빠트리면 런타임에 문제. <br>
인스턴스의 타입만으로는 현재 나타내는 의미를 알 길이 없다. <br>

>태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율적 <br>

>서브타이핑(subtyping) : 클래스 계층구조를 활용하는 수단. <br>

---

### 태그 달린 클래스를 클래스 계층구조로 바꾸는 방법
1. 계층구조의 루트(root)가 될 추상 클래스 정의
	- 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 선언
	- 예제 코드의 Figure 클래스에서는 area가 이러한 메서드에 해당
2. 태그 값에 상관없이 동작이 일정한 메서드들을 루트 클래스에 일반 메서드로 추가
	- 모든 하위 클래스에서 공통으로 사용하는 데이터 필드들도 전부 루트 클래스로 올린다.
    - Figure 클래스에서는 태그 값에 상관없는 메서드가 하나도 없고, 모든 하위 클래스에서 사용하는 공통 데이터 필드도 없다.
    - 그 결과 루트 클래스에는 추상 메스드인 area 하나만 남는다.
3. 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의
	- Figure를 확장한 원(Circle) 클래스와 사각형(Rectangle) 클래스 만들면 된다.
    - 각 하위 클래스에는 각자의 의미에 해당하는 데이터 필드들을 넣는다.
    - 원에는 반지름(radius), 사각형에는 길이(length), 너비(width)를 넣으면 된다.
4. 루트 클래스가 정의한 추상 메서드를 각자의 의미에 맞게 구현

```java
abstract class Figure {
	abstract double area();
}

class Circle extends Figure {
	final double radius;
    
    Circle(double radius) { this.radius = radius; }
    
    @Override double area() { return Math.PI * (radius * radius); }
}

class Rectangle extends Figure {
	final double length;
    final double width;
    
    Rectangle(double length, double width) {
    	this.length = length;
        this.width = width;
    }
    
    @Override double area() { return length * width; }
}
```

-> 살아남은 피드들 모두 final. <br>
-> 루트 클래스의 코드를 건드리지 않고도 다른 프로그래머들이 독립적으로 계층구조를 확장하고 함께 사용 가능. <br>
-> 타입이 의미별로 따로 존재하나 변수의 의미를 명시하거나 제한할 수 있고, <br>
-> 특정 의미만 매개변수로 받을 수 있음 <br>
-> 유연성과 컴파일타임 타입 검사 능력을 높여줌 <br>


>태그 달린 클래스를 써야 하는 상황은 거의 없다. <br>
새로운 클래스를 작성하는 데 태그 필드가 등장한다면 <br>
태그를 없애고 계층구조로 대체하는 방법을 생각해보자. <br>
기존 클래스가 태그 필드를 사용하고 있다면 계층구조로 리팩터링하는 걸 고민해보자.
