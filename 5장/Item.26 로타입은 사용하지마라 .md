### 제네릭 타입(제네릭 클래스, 제네릭 인터페이스)

```
List<Integer> numbers = new ArrayList<>();
```

- 클래스와 인터페이스 선언에 `타입 매개변수`를 사용한 클래스와 인터페이스
- List → E는 타입 매개변수
- 제네릭은 JDK 1.5부터 지원됨

### 로 타입

```
List a = new ArrayList();
```

- 제네릭 타입에서 타입 매개변수를 전혀 사용하지 않는 타입
- 타입 정보가 전부 지워진 것처럼 동작함

### 로 타입의 문제

```
//삽입과정
List stringCollection = new ArrayList(); // String을 넣으려고 만든 컬렉션
stringCollection.add("1");
stringCollection.add("2");
// 이런 저런 문자열 값들이 들어다가
stringCollection.add(2); // 컴파일 오류 x
```

타입 매개변수가 없기 때문에 어떤 타입들을 넣어도 `컴파일에러`, `런타임 에러`가 발생하지 않는다.

```
for (Iterator i = stringCollection.iterator(); i.hasNext(); ) {
    String s = (String) i.next();
    //필요한 작업
}
```

하지만 컬렉션에 담은 값들을 꺼내는 과정에서 String을 담는 컬렉션을 의도했기 때문에 
(String)키워드를 통해 `형변환`을 하는데 위에서 넣는 2라는 값 때문에, `ClassCastException`이 발생한다.

추가적으로 해당 컬렉션에 이곳 저곳에서 쓰인다면, 어디서 잘못된 값이 들어갔는지 찾기가 힘들 것이다.

### 로타입을 쓰면 제네릭이 안겨주는 안정성과 표현력을 모두 잃게 된다.

### 제네릭 타입을 쓰면 컴파일 단계에서 타입 안정성을 가져갈 수 있다.

```
List<String> stringCollection = new ArrayList<>();
stringCollection.add("1");
stringCollection.add("2");
stringCollection.add(2); // 컴파일 에러 발생!
```

`다른 타입의 인스턴스`가 들어오면 `컴파일 에러`를 발생시킨다.

### 제네릭 타입 쓰면 되지 그럼 로타입은 대체 왜 허용되는가?

사람들이 제네릭을 받아들이는데 오랜 시간이 걸렸고, 이미 로타입으로 작성된 코드들이 너무 많기 때문에, 
그 코드들과의 호환성을 위해서 남겨두었다.

### 모든 타입을 허용하려면 로타입이 아닌 Object 타입으로 명시하여 사용하라

로 타입은 제네릭 타입에서 아예 **발을 뺀** 것이고Object 제네릭 타입은
**모든 타입을 허용한다는 것을 컴파일러에서 `명시`**한 것이다.

```
List stringCollection = new ArrayList(); //로타입 나 몰라라
stringCollection.add(1);
stringCollection.add("2");
```

```
List<Object> stringCollection = new ArrayList<>(); //난 모든 타입을 받을거야!!
stringCollection.add(1);
stringCollection.add("2");
```

하지만 모든 타입에 대한 허용을 하려고 메소드의 매개 변수에 `List<Object>`타입을 사용하면 문제가 있다.

```
public static void main(String[] args) {
    List<String> stringCollection = new ArrayList<>();
    rawTypeMethod(stringCollection); // 컴파일 에러 발생x
    objectTypeMethod(stringCollection); //컴파일 에러
}

private static void rawTypeMethod(final List stringCollection) {}

private static void objectTypeMethod(final List<Object> stringCollection) {}
```

로 타입 List 는 `List<String>`의 상위 타입이기 때문에 rawTypeMethod는 컴파일 에러가 발생하지 않는다하지만
**로타입은 위에서도 살펴봤듯이 런타임 에러가 발생할 상황이 농후하기 때문에 불안정하기 때문에 사용하지 말자.**

모든 타입을 받으려고 선언한 `List<Object>`는 제네릭의 `하위 타입 규칙` 때문에 `List<String>`을 받지 못하고
컴파일 에러가 발생한다. Object와 String이 부모 자식 관계라서 될 것 같지만,
`List<Object>`에는 어떤 타입의 값도 넣을 수 있지만 `List<String>`은 문자열만 넣을 수 있다.
`List<String>`을 `List<Object>` 하위 타입이라고 생각한다면`List<String>`은
자신의 상위 타입 `List<Object>`의 모든 일을 할 수 없기 때문에 리스코프 치환 원칙에 어긋난다.

### 비한정 와일드카드 타입

위의 제약을 없애기 위해 모든 타입을 받을 수 있는 List<?>와일드카드를 사용하면 된다.

```
public static void main(String[] args) {
        List<String> stringCollection = new ArrayList<>();
        wildCardTypeMethod(stringCollection); // 컴파일에러 발생x
    }

    private static void wildCardTypeMethod(final List<?> stringCollection) {
        stringCollection.get(0);
        stringCollection.add(1); // 컴파일 에러
        stringCollection.add("1"); // 컴파일 에러
    }
```

하지만 여기에도 제약이 생긴다. 와일드 카드로 모든 타입들을 받을 수 있지만,
**`타입 불변식`을 훼손하지 못하게 null외의 아무 값도 넣지 못한다.**

main에서 stringCollection은 `List<String>`인데, 메소드의 매개변수는 `List<?>`이다.`List<?>`에선, 
모든 제네릭을 다 받을 수 있지만, 해당 컬렉션 객체가 어떤 타입의 제네릭이었는지 알 수 가 없기 때문에 add를 하지 못하게 한다. 
**잘못된 타입의 값을 넣었다가 타입 불변식을 훼손할 수 있기 때문**이다.

하지만 get과 같이 타입 불변을 훼손하지 않는 행동은 할 수 있다.

이에 반해 로 타입 List은 **아무 값이나 add** 할 수 있기 때문에 **타입 불변식을 훼손할 수 있다.**

### 로타입은 타입 불변식을 훼손할 수 있기 때문에 사용하지 말자.

하지만 로 타입을 써야하는 예외가 있다.

### 로 타입을 써야하는 예외 1 - class 리터럴

- class 리터럴에는 배열과 기본타입은 허용하지만 **매개변수화 타입을 사용할 수 없다.**
- `List.class` 허용`List<String>.class` 허용X

### 로 타입을 써야하는 예외 2 - instanceof 연산자

- 런타임에는 매개변수화 정보가 지워진다. 컴파일 단계에선 잘못된 타입에 대한 체크가 끝나기 때문
- `instanceof List`와 `instanceof List<?>`은 똑같이 동작한다.![]
