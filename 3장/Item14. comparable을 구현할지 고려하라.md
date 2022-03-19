## 아이템 14. comparable을 구현할지 고려하라

primitive 타입의 경우 부등호를 갖고 쉽게 두 변수를 비교할 수 있었다. 자바 자체에서 제공 되기에 별다른 처리 없이 비교가 가능. 그러나 새로운 클래스 객체를 만들어 비교하고자 할때 어떻게 될까?

예를들어 학생의 나이와 학급 정보를 갖고있는 클래스를 만든다고 가정해보자. a,b학생 두 객체를 생성했을 때, 어떻게 비교할 것인가? 나이? 학급정보?

객체는 사용자가 기준을 정해주지 않는 이상 어떤 객체가 더 높은 우선순위를 갖는지 판단할 수 가 없다. 이러한 문제점을 해결하기 위해 바로 comparable , comparator가 쓰이는 것이다.

<br>

## Comparable

기본적인 정렬(오름차순, 사전순)을 구현할 때 사용

Java에서 제공하는 정렬 가능한 클래스들은 모두 Comparable 인터페이스를 구현하고 있음

정렬할 객체에 Comparable인터페이스를 implements 한 후, compareTo()메서드를 오버라이드 하여 구현한다.

자기 자신과 매개변수 객체를 비교

comparable의 유일무이한 메서드는 compareTo.

```java
public interface Comparable<T> {
	int compareTo(T t);
}
```

<br>

**compareTo 규약을 지키지 못하면 비교를 활용하는 클래스와 어울리지 못한다.**

compareTo 규약

1. 두 객체 참조의순서를 바꿔 비교해도 예상한 결과가 나와야 한다. (1<2 =2>1 / 1=2 = 2=1 / 1>2 = 2<1)
2. 첫 번째가 두 번째보다 크고 두번째가 세번째 보다 크면, 첫번재는 세번째 보다 커야한다
3. 크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상같아야 한다.

<br>

compareTo 메소드 작성법

현재객체 < 파라미터로 넘어온 객체 : 음수 리턴

현재객체 == 파라미터로 넘어온 객체 : 0 리턴

현재객체 > 파라미터로 넘어온 객체 : 양수 리턴

음수 또는 0이면 객체의 자리가 그대로 유지

양수일경우 두 객체의 자리가 바뀐다.

<br>

 compareTo 메서드에서 필드의 값을 비교할 때 < 와 >연산자는 쓰지 말아야 한다. 대신 박싱된 기본타입 클래스가 제공하는 정적 compare메서드나 Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용

<br>

## Comparator

기본적이지 않은 정렬(내림차순,사전역순,여러개의 기준으로 정렬) 할때 사용

두 매개변수 객체를 비교

메소드가 많지만(자바의 숫자용 기본타입을 모두 커버) 실질적으로 구현하는건 compare

```java
public interface Comparator<T> {
	int compareTo(T t1,T t2);
}
```

<br>

Comparable의 compareTo는 자기 자신과 매개변수를 비교한다고 했고, compareTo는 정수를 반환하며, 자기 자신을 기준으로 상대방과의 차이 값을 비교하여 반환한다고 했다.

이를 좀 더 생각해보면 -1, 0, 1로 반환할 수도 있으나, 그냥 두 비교대상의 값 차이를 반환해도 될듯?

```java
static Comparator<T> hashCodeOrder = new Comparator<>() {
	public int compare(T t1,T t2) {
		return t1.hashCode() - t2.hashCode();
	}
};
```

그러나 **이 방식은 사용하면 안된다. 정수 오버플로 등 오류 발생**

<br>

```java
static Comparator<T> hashCodeOrder = new Comparator<>() {
	public int compare(T t1,T t2) {
		return Integer.compare(t1.hashCode(), t2.hashCode());
	}
};
```

ㄴ 정적 compare메서드를 활용한 비교자

```java
static Comparator<T> hashCodeOrder = 
	Comparator.comparingInt(t -> t.hashCode());
```

ㄴ 비교자 생성 메서드를 활용한 비교자
