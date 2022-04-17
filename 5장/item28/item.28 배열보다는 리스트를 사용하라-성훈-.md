# item28. 배열보다는 리스트를 사용하라 

---

## 배열과 제네릭

### 배열:

* 공변
* sub가 Super 하위 타입이면 배열 Sub[]는 배열 Super[]의 하위 타입이 된다.
* 함께 변한다.

 
```
Super sub = new Sub();
```
배열에서 공변은 아래처럼 배열에서도 함께 변해서 적용된다

```
Super[] sub = new Sub[]{};
```

ex)
```
Object[] objects = new String[1]; // 배열은 공변하므로 String[]은 Object[]의 하위 타입이므로 컴파일 가능
objects[0] = 1;
```
* objcets는 컴파일 타임에 Object[]이므로 Integer을 할당할 수 있으나, 런타임엔 String[]이기 때문에 예외가 발생합니다.


>자바를 사용하는 이유 중 하나가 정적 컴파일 언어로 시스템의 안정성을 높여주기 위함인데 배열의 이러한 특징은 컴파일 타임에 타입 안전성을 보장해줄 수 없다

### 제네릭:

* 불공변

* Sub가 Super의 하위 타입이더라도 `ArrayList<Super>`
  는 `ArrayList<Super>` 의 하위 타입이 아닙

```
List<Super> subs = new ArrayList<Sub>(); // 같은 타입이 아니므로 컴파일 에러!
```
위 코드처럼 Sub가 Super를 상속하더라도 같은 타입이 아니면 컴파일 에러가 뜸

ex)

```
ArrayList<Object> objectList = new ArrayList<String>(); // 제네릭 타입은 불공변하므로 컴파일 불가능
```
* 제네릭 타입은 불공변하기 때문에 String 타입이 Object 타입을 확장했다 하더라도 컴파일 에러가 발생
* 만약 제네릭 타입이 공변하여 컴파일이 된다면 배열과 동일하게 런타임에 타입 에러가 발생
* 그러므로 배열 대신 제네릭 타입을 사용하는 것이 타입 안전한 프로그래밍을 할 수 있다


## 실체화와 소거

### 배열은 런타임에 실체화
```
Object[] objects = new String[1];
```
* 배열은 런타임에 타입이 실체화 되기 때문에 objects는 런타임에 String[]가 된다.

### 제네릭 타입은 런타임에 소거

```
// 컴파일 타임(실제 작성한 코드)
ArrayList<String> stringList = new ArrayList<String>();
ArrayList<Integer> integerList = new ArrayList<Integer>();

// 런타임(제네릭 타입은 런타임에 소거되므로 구분이 불가능하다)
ArrayList stringList = new ArrayList();
ArrayList integerList = new ArrayList();
``` 
* 제네릭 타입은 런타임에 소거되므로 런타임에는 타입이 소거된 ArrayList만 남게된다.

* 배열은 런타임 시점에 타입을 인지하고 실행하는데(실체화) 반해 제네릭은 런타임 시점에 타입 정보를 소거(erasure)한다.

* 제네릭의 타입 시스템 자체가 컴파일 시점에 에러를 잡고 런타임에 ClassCastException이 발생하는 일을 막아주겠다는 뜻이기 때문.

* 배열과 제네릭은 그 자체로 잘 어우러지지 못한다.

* 제네릭 배열을 직접 생성할 순 없지만 와일드카드 타입을 이용하거나, 강제 형변환을 통해 제네릭 배열을 사용할 수 있습니다

---

>`핵심:`
배열은 공변하며 런타임에 실체화 되지만, 제네릭 타입은 불공변하며 런타임에 소거
>
>이로 인해 배열은 타입 안전성을 보장해줄 수 없어 제네릭 배열을 직접 생성할 수 없다
>
>타입 안전성을 위해서라면 배열을 사용하기 보다 제네릭 타입을 활용한 리스트를 사용하는 것이 좋음.   
@타입에 더 안전하며 런타임이 아닌 컴파일 시점에 에러를 잡아주기 때문@




