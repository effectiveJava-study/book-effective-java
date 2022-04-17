# 아이템 33

# 타입 안전 이종 컨테이너를 고려하라

제네릭은 `Set<E>`, `Map<K,V>` 등의 컬렉션과`ThreadLocal<T>` , `AtomicReference<T>` 등의 단일원소

컨테이너에도 흔히 쓰인다고 한다. 이런 모든 쓰임에는 매개변수 대상은 컨테이너 자신이기 때문에

하나의 컨테이너에서 매개변수화할 수 있는 타입의 수가 제한된다. 컨테이너의 일반적인 용도에 맞

게 설계된 것이기 때문에 문제될 것은 없다. 예컨데 Set에는 원소 타입을 뜻하는 단 하나의 타입 매개

변수만 있으면 되며, Map에는 키와 값의 타입을 뜻하는 2개만 필요한 식이다.

### 유연한 수단의 컨테이너가 필요할 때가 종종 있다.

타입의 수에 제약없이 유연하게 필요한 경우, 특정 타입 외에 다양한 타입을 지원해야하는 경우가 

있을 수 있다.

### 타입 안전 이종 컨테이너 패턴

위 고민에 대한 해법은  컨테이너 대신 키를 매개변수화한 다음 컨테이너에 값을 넣거나, 뺄때 

키 타입을 제공해주면 된다. 이렇게 하면 제네릭 타입 시스템이 값의 타입이 키와 같음을 보장해 

줄 것이다. 이러한 설계 방식을 ***타입 안전 이종 컨테이너 패턴***이라고 한다.

```java
public class Favorites{
        public <T> void putFavorite(Class<T> type, T instance);
        public <T> getFavorite(Class<T> type);
    }
```

다음 main은 Favorites 클래스를 사용하고 있는 예시이다. String, Integer, Class 인스턴스를 저장, 

검색, 출력하고 있다.

```java
public static void main( final String[] args ){
       Favorites f = new Favorites();
       
       f.putFavorite(String.class, "java");
       f.putFavorite(Integer.class, 0xcafebabe);
       f.putFavorite(Class.class, Favorites.class);
       
       String favoriteString = f.getFavorite(String.class);
       int favoriteInteger = f.getFavorite(Integer.class);
       Class<?> favoriteClass = f.getFavorite(Class.class);
       
       System.out.printf("%s %x %s %n", favoriteString, favoriteInteger, favoriteClass.getName());
    }
```

이 방식이 동작하는 이유는 class의 클래스가 제네릭이기 때문이다.

class [리터럴](https://mine-it-record.tistory.com/100) 타입은 Class가 아닌 `Class<T>` 이다.

String.class의 타입은 `Class<String>` 이고, Integer.class의 타입은 `Class<Integer>` 인 것이다.

컴파일타임 타입 정보와 런타임 타입 정보를 위해 메소드에서 주고 받는 class 리터럴을

**타입 토큰** 이라고 한다.

타입 토큰(Type Token)은 쉽게 말해 타입을 나타내는 토큰이며, 클래스 리터럴이 타입 토큰으로서 

사용된다.

(ex. Integer.class, String.class 는 class 리터럴이며 타입토큰은 `Class<Integer>`, `Class<String>`)

타입 토큰은 타입 안전성이 필요한 곳에 주로 사용된다.

### 핵심 정리

1. 컬렉션 API로 대표되는 일반적인 제네릭 형태에서는 한 컨테이너가 다룰 수 있는 타입의 매개변수는 고정되어 있다.
2. 키를 타입 매개변수로 바꾸면 이런 제약이 없는 타입 안전 이종 컨테이너를 만들 수 있다.
3. Class를 키로 쓰며, 이런 식으로 쓰이는 Class 객체를 타입 토큰이라고 한다.
