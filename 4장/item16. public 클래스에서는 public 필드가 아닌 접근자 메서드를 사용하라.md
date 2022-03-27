
## 16 public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라.

필드가 public으로 선언돼 있으면 데이터 필드에 직접 접근 가능해 캡슐화의 이점이 사라진다.
```
class Point{
    public double x;
    public double y;
}
```
여기서 필드를 모두 private으로 바꾸고 접근자를 사용해보면
```
class Point{
    public double x;
    public double y;

    public Point(double x, double y){
        this.x = x;
        this.y = y;
    }

    public double getX(){return x;}
    public double getY(){return y;}

    public void setX(double x){this.x = x;}
    public void setY(double y){this.y = y;}
}
```
public클래스라면 이 방식이 맞다. 패키지 밖에서 접근할 수 있는 클래스라면 접근자를 제공해 내부 표현 방식을 바꿀 수 있는 유연성을 얻을 수 있다.

하지만 default 혹은 private 중첩 클래스라면 데이터 필드를 노출해도 문제가 없다. 그 클래스가 표현하려는 추상 개념만 올바르게 표현해주면 된다.

package-private 클래스
```
class Point {
	public double x;
	public double y;

}
```
선언 면이나 이를 사용하는 클라이언트 코드 면에서나 접근자 방식보다 훨씬 깔끔하다. 클라이언트 코드가 이 클래스 내부 표현에 묶이지만, 클라이언트도 어차피 이 클래스를 포함하는
패키지 안에서만 동작하는 코드일 뿐이다. 다시 말해 패키지 내에서만 바꿔주면 된다.

```
class Point {
	private double x;
	private double y;

    public Point (double x, double y){
    	this.x = x;
        this.y = y;
    }

    public void change (double x, double y){
    	this.x = x;
        this.y = y;
    }
}
```
수정 범위가 더 좁아져서 이 클래스를 포함하는 외부 클래스까지 제한된다.

자바 플랫폼에도 public 클래스의 필드를 직접 노출하지 말라는 규칙을 어기는 사례가 종종 있다. 이를 흉내 내지 말고 타산지석으로 삼아야 한다.
그 예가 java.awt.package의 Point와 Dimension 클래스다.

public 클래스가 불변이라면 직접 노출할 때의 단점이 조금 줄어들지만 API를 변경하지 않고는 표현 방식을 바꿀 수 없고 필드를 읽을 때 부수 작업을 수행할 수 없다는
단점은 여전하다. 그래도 불변식은 보장해준다.
```
public final class Time { 
    private static final int HOURS_PER_DAY = 24;
    private static final int MINUTES_PER_HOUR = 60;

    public final int hour;
    public final int minute;  

    public Time(int hour, int minute) {
        if(hour < 0 || hour >= HOURS_PER_DAY) 
            throw new IllegalArgumentException("시간: " + hour);
        if(hour < 0 || minute >= MINUTES_PER_HOUR)
            throw new IllegalArgumentException("분: " + minute);
        this.hour = hour;
        this.minute = minute; 
        // ... 나머지 코드 생략
    }
}
```

결론: public 클래스는 절대 가변 필드를 직접 노출해서는 안된다. 불변 필드라면 노출해도 덜 위험하지만 완전히 안심할 수는 없다. 하지만 default나 private 중첩 클래스는 종종 필드를 노출
하는 편이 나을 때도 있다.

