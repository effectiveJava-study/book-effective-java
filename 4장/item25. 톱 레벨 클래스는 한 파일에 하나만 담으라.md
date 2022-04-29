소스파일 하나에 톱레벨 클래스를 여러개 선언 하여도 자바 컴파일러는 불평하지 않는다.

*But, 아무런 득이 없고 심각한 위험을 감수 해야 하는 행위*

: 한 클래스를 여러가지로 정의, 그로 인한 어느 소스파일을 먼저 컴파일하는지에 따라 달라짐.

```java
public class Main {
	public static void main(Strin[] args) {
		System.out.println(Utensil.NAME + Dessert.NAME);
	}
}
```
Main클래스는 다른 톱레벨 클래스 2개 (Utensil, Dessert)를 참조

```java
class Utensil {
	static final String NAME = "pan";
}

class Dessert {
	static final String NAME = "cake";
}
```
두 클래스가 한 파일에 정의 됨(Utensil.java)

```java
class Utensil {
	static final String NAME = "pot";
}

class Dessert {
	static final String NAME = "pie";
}
```
두 클래스가 한 파일에 정의 됨(Dessert.java)

**javac Main.java / javac Main.java Utensil.java** : pan cake가 출력

**javac Main.java Dessert.java**: 

운 좋게 javac Main.java Dessert.java 명령으로 컴파일 하면 컴파일 오류가 나고 중복정의 했다고 알려줌.

**javac Dessert.java Main.java** : potpie가 출력

컴파일러에 어느 소스 파일을 먼저 건네느냐에 따라 동작이 달라져 반드시 바로잡아야 할 문제이다.

**해결책**

톱레벨 클래스들을 서로 다른 소스 파일로 분리!

굳이 여러 톱레벨 클래스를 한 파일에 담고 싶다면 정적 멤버 클래스를 사용하는 방법을 고민.

```java
public class Test {
	public static void main(String[] args) {
		System.out.println(Utensil.NAME + Dessert.NAME);
	}

	private static class Utensil {
		static final String NAME = "pan";
	}

	private static class Dessert {
		static final String NAME = "cake";
	}
}
```
