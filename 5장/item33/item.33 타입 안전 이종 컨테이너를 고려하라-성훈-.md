# item.33 타입 안전 이종 컨테이너를 고려하라



## type safe heterogeneous container:

* type-safe (타입 안전) : generic을 통해 타입 안전성이 보장.

* hetrogeneous (이종) : 서로 다른 타입이 하나의 컨테이너에 존재할 수 있음을 뜻함.

* container (컨테이너) : 무언가를 담고 있는 객체를 뜻한다. 쉽게 Map이나 Set등의 컬렉션이나 박싱 클래스 등을 예로 들 수 있다.

**여러 다른 종류들로 이루어진 값을 저장하는 타입에 안전한 객체**

## 사용하는 이유
하나의 컨테이너에서 매개변수화할 수 있는 타입의 수가 제한된다

Set<E>, Map<K,V> 처럼 클래스 레벨에서 매개변수화 할 수 있는 타입의 수는 제한적이다. (ex. Map 은 2개)


따라서 파라미터화된 제네릭 Map<String, Integer>가 있다면 Integer 이외의 값을 담을 수 없다. 이렇게 제네릭 타입의 수가 정해져 있는 것이 일반적이고, 실제로 문제 없이 작동한다.


`하지만 이 Map에 Integer뿐 아니라 String, Double, Boolean 타입도 함께 답고 싶을 때는 어떻게 할까?`

이때 사용하는 것이 타입 안전 이종 컨테이너
(서로 다른 타입을 하나의 컨테이너에 안전하게 보관)

## 사용방법

컨테이너를 매개변수화 하지 않고 컨테이너의 키를 매개변수화 하면 된다.

Map<String, Integer> : 컨테이너를 매개변수화 했다.

Map<Class<?>, Object> : 컨테이너의 키를 매개변수화 했다.

#### 타입별로 즐겨 찾는 인스턴스를 저장하고 검색할 수 있는 Favorites 클래스


```
private Map<Class<?>, Object> favorites = new HashMap<>();

public <T> void putFavorites(Class<T> type, T instance) {
    favorites.put(Objects.requireNonNull(type), instance);
}

public <T> T getFavorite(Class<T> type) {
    return type.cast(favorites.get(type));
}

```
## 이유

이러한 구조가 가능한 것은 Class클래스가 제네릭이기 때문.
이 favorites 맵의 키에는 class 리터럴이 들어가고, 값으로는 Object타입이 들어가기 때문.

`class 리터럴`
>리터럴이란 변수에 할당되는 상수를 뜻한다.
>
>class 리터럴은 Class타입의 상수를 뜻하며, 그 예로는 String.class, Integer.class, Member.class 등이 있다.
>
>특히 이 중, 제네릭에서 매개변수화된 타입으로 쓰일 때 이를 타입 토큰이라 한다.

여기서 벨류 타입이 Object이기 때문에 모든 타입이 올 수 있다


**`핵심`**
* 컬렉션 API로 대표되는 일반적인 제네릭 형태에서는 한 컨테이너가 다룰 수 있는 타입매개변수의 수가 고정되어 있다.

* `하지만 컨테이너 자체가 아닌 키를 타입 매개변수로 바꾸면 이런 제약이 없는 타입 이종 컨테이너를 만들 수 있다.`

* 타입 안전 이종 컨테이너는 Class를 키로 쓰며, 이런 식으로 쓰이는 Class 객체를 타입 토큰이라 한다.


