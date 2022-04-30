# item. 41 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라

## 마커 인터페이스:
>마커 인터페이스(marker interface)란, 일반적인 인터페이스와 동일하지만, 아무 메서드도 선언하지 않은 인터페이스이다. 자바의 대표적인 마커 인터페이스로는 Serializable, Cloneable, EventListener가 있다. 대부분의 경우 마커 인터페이스를 단순한 타입 체크를 하기 위해 사용한다

* Serializable
```java
package java.io;

  public interface Serializable {
  }

```
Serializable 인터페이스를 구현한 클래스는 ObjectOutputStream을 통해 직렬화할 수 있다.

* Item : Serializable을 구현하지 않음
```java
import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.Setter;

import java.math.BigDecimal;

@Getter
@Setter
@AllArgsConstructor
public class Item {
    private long id;
    private String name;
    private BigDecimal price;
}
```

```java
public class SerializableTest {

    @Test
    void serializableTest() throws IOException {

        File f= new File("test.txt");
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream(f));
        objectOutputStream.writeObject(new Item(1L, "item A", new BigDecimal(30000)));
    }
}

----------------------------------

java.io.NotSerializableException: ch6.dahye.item41.Item

    at java.base/java.io.ObjectOutputStream.writeObject0(ObjectOutputStream.java:1185)
    at java.base/java.io.ObjectOutputStream.writeObject(ObjectOutputStream.java:349)

```
Serializable 을 구현하지 않은 객체를 직렬화하려고하면 다음과 같이 오류가 발생
Serializable을 구현하지 않은 경우 NotSerializableException 예외를 발생하고 있다



```java

if (obj instanceof String) {
                writeString((String) obj, unshared);
            } else if (cl.isArray()) {
                writeArray(obj, desc, unshared);
            } else if (obj instanceof Enum) {
                writeEnum((Enum<?>) obj, desc, unshared);
            } else if (obj instanceof Serializable) {
                writeOrdinaryObject(obj, desc, unshared);
            } else {
                if (extendedDebugInfo) {
                    throw new NotSerializableException(
                        cl.getName() + "\n" + debugInfoStack.toString());
                } else {
                    throw new NotSerializableException(cl.getName());
                }
            } 
          
```
단순히 타입 체크 정도로 사용

>직렬화:     
>생성한 객체를 파일로 저장할 일이 있을 수도 있다   
저장한 객체를 읽을 일이 생길 수도 있다.   
다른 서버에서 생성한 객체를 받을 일도 생길 수 있다.   
>
>자바 시스템 내부에서 사용되는 객체 또는 데이터를 외부의 자바 시스템에서도 사용할 수 있도록 바이트(byte) 형태로 데이터 변환하는 기술과 바이트로 변환된 데이터를 다시 객체로 변환하는 기술
>
>시스템적으로 이야기하자면 JVM(Java Virtual Machine 이하 JVM)의 메모리에 상주(힙 또는 스택)되어 있는 객체 데이터를 바이트 형태로 변환하는 기술과 직렬화된 바이트 형태의 데이터를 객체로 변환해서 JVM으로 상주시키는 형태





## 마커 어노테이션
* 데이터로 구성되지 않고 주 목적은 어노테이션 선언을 표시하는 것
* 해당 요소가 특정 속성을 가짐을 나타내는 어노테이션
* 대표적으로 @Override 가 있음
* 어노테이션에 별다른 멤버를 추가하지 않아도,
 메서드가 오버라이딩 조건을 충족하는지 검증을 해준다.

일반적인 인터페이스와 동일하지만 사실상 아무 메소드도 선언하지 않은 인터페이스

## 마커 인터페이스가 마크 어노테이션보다 좋은점

### 1. 마커 인터페이스는 타입을 정의할 수 있지만 마크 어노테이션은 그렇지 않다.

>마커 인터페이스는 타입으로 사용하여 컴파일타임 에 오류를 검출할 수 있다.
마커 에너테이션은 런타임에서야 오류를 검출할 수 있다.



### 2. 마커 인터페이스는 적용 대상을 더 정밀하게 지정할 수 있다
> 마커 에너테이션은 @Retention(RetentionPolicy.TYPE) 에너테이션을 통해 클래스, 인터페이스, enum, 에너테이션에만 달 수 있는 마커를 만들 수 있다.
하지만 마커 인터페이스는 이보다 더 정밀한 제한을 둘 수 있다.


## 마커 어노테이션이 마커 인터페이스 보다 좋은점

### 거대한 애너테이션 시스템의 지원을 받을 수 있다.
> 스프링부트와 같은 어노테이션 기반의 프레임워크에서 일관성을 유지할 수 있다.
>
>필드나 메서드에는 마크 인터페이스를 사용할 수 없으니, 마커 어노테이션을 사용해야 한다.




# 핵심

* 언제 마커 어노테이션을 사용할까? 클래스나 인터페이스를 제외한 프로그램 요소(필드나 메서드)

* 언제 마커 인터페이스를 사용할까? 마커가 되는 타입이 파라미터에 사용되어 컴파일 타임에 안정성이 필요할 때

> 마커 인터페이스와 마커 어노테이션은 각자의 쓰임이 있는데,
새로 추가하는 메서드 없이 단지 타입 정의가 목적이라면 마커 인터페이스를 선택하자.


