# item. 37 ordinal 인덱싱 대신 EnumMap을 사용하라


### ordinal() 메소드
>ordinal() 메소드는 해당 열거체 상수가 열거체 정의에서 정의된 순서(0부터 시작)를 반환.
>
>이때 반환되는 값은 열거체 정의에서 해당 열거체 상수가 정의된 순서이며, 상숫값 자체가 아님

### EnumMap 
 >Map 인터페이스에서 키를 특정 enum 타입만을 사용하도록 하는 구현체.
>
>EnumMap은 내부에 데이터를 Array에 저장한다
>
>장점은 성능. 그리고 특정 enum 타입만을 키로 받기 때문에 인터페이스가 명확해 진다

## ordinal 기반 인덱싱 

배열이나 리스트에서 원소를 꺼낼 때 ordinal 메서드로 인덱스를 얻는 코드가 있다.
식물의 생애주기를 열거 타입으로 표현한 LifeCycle 열거 타입을 예로 보자

 
```java

 public class Plant {

    enum LifeCycle { ANNUAL, PERNNIAL, BIENNIAL}

    final String name;
    final LifeCycle lifeCycle;

    public Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override
    public String toString() {
        return name;
    }
}
```
 이제 정원에 심은 식물들을 배열 하나로 관리하고, 이들을 생애주기(한해살이, 여러해살이, 두해살이)별로 묶어보자.

 생애주기별로 총 3개의 집합을 만들고 정원을 한 바퀴 돌며 각 식물을 해당 집합에 넣는다. 이때 어떤 프로그래머는 집합들을 배혈 하나 넣고 생애주기의 ordinal 값을 그 배열의 인덱스로 사용

 ```java

public static void usingOrdinalArray(List<Plant> garden) {
    Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[LifeCycle.values().length];
    for (int i = 0 ; i < plantsByLifeCycle.length ; i++) {
        plantsByLifeCycle[i] = new HashSet<>();
    }

    for (Plant plant : garden) {
        plantsByLifeCycle[plant.lifeCycle.ordinal()].add(plant);
    }

    for (int i = 0 ; i < plantsByLifeCycle.length ; i++) {
        System.out.printf("%s : %s%n", LifeCycle.values()[i], plantsByLifeCycle[i]);
    }
}
```
1. Set 배열을 생성해 생애주별로 관리함, 총 3개의 배열이 만들어지고, 각 배열을 순회하여 빈 HashSet으로 초기화 한다.

2. plant 들을 배열의 Set에 추가하고 이때 plant가 가지고있는 LifeCycle 열거타입의 ordinal 값으로 배열의 인덱스를 결정, 그결과 식물의 생애주기 별로 Set 추가된다.

3. 결과를 출력할 때 열거 타입의 values로 반환되는 열거 타입 상수 배열의 순서는 ordinal 값으로 결정되개 때문에 Set 배열의 각 Set이 의미하는 생애주기는 values 의 순서와 같을것이다.

<해당 코드는 여러가지 문제가 있다>
* 배열은 제네릭과 호환되지 않아서 비검사 형변환을 수행(아이템28)해야 하고 깔끔히 컴파일되지 않는다.

* 사실상 배열은 각 인덱스가 의미하는 바를 알지 못하기 때문에 출력 결과에 직접 레이블을 달아야 한다.

* 정수는 열거 타입과 달리 안전하지 않기 때문에 정확한 정숫값을 사용한다는 것을 직접 보증해야 한다.

>잘못된 값을 사용하면 잘못된 동작을 묵묵히 수행하거나 (운이좋으면)
ArrayIndexOutOfBoundsException을 볼 수 있음

## EnumMap
이러한 단점들을 해결하기 위해서 나온것이 EnumMap.
EnumMap은 열거 타입을 키로 사용하는 Map 구현체이다.

 ```java
public static void usingEnumMap(List<Plant> garden) {
    Map<LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(LifeCycle.class);

    for (LifeCycle lifeCycle : LifeCycle.values()) {
        plantsByLifeCycle.put(lifeCycle,new HashSet<>());
    }

    for (Plant plant : garden) {
        plantsByLifeCycle.get(plant.lifeCycle).add(plant);
    }

    //EnumMap은 toString을 재정의하였다.
    System.out.println(plantsByLifeCycle);
}
```
1. 아까 ordinal을 사용한 코드와 다르게 안전하지 않은 형변환을 쓰지 않음.

2. EnumMap 자체가 toString 을 제공하기 때문에 출력 결과에 레이블을 달 일도 없다.

3. ordinal를 이용한 배열 인덱스를 사용하지 않아 인덱스를 계산하는 과정에 오류가 날 가능성도 없어짐.

4. **EnumMap은 그 내부에서 배열을 사용하기 때문에 내부 구현 방식을 안으로 숨겨서 Map의 타입 안정성과 배열의 성능을 모두 얻어냈다.**

여기서 EnumMap의 생성자가 받는 키 타입의 Class 객체는 한정적 타입의 토큰으로, 런타임 제네릭 타입 정보를 제공한다(아이템33). 스트림(아이템45)을 사용하면 코드를 더 줄일 수 있다. 


 ```java

public static void streamV1(List<Plant> garden) {
    Map plantsByLifeCycle = garden
    .stream()
    .collect(Collectors
    .groupingBy(plant -> plant.lifeCycle));
    System.out.println(plantsByLifeCycle);
}

public static void streamV2(List<Plant> garden) {
    Map plantsByLifeCycle = garden
    .stream()
    .collect(Collectors.groupingBy(plant -> plant.lifeCycle,
                    () -> new EnumMap<>(LifeCycle.class)
                    ,Collectors.toSet()));
    System.out.println(plantsByLifeCycle);
}
 ```

Collectors의 groupingBy 메소드를 이용하여 맵을 구성하였는데, v1과 v2의 차이는 groupingBy 메소드에 원하는 맵 구현체를 명시하였나 차이다.

v1 메소드는 EnumMap이 아닌 고유한 맵 구현체를 사용해서 EnumMap을 써서 얻는 공간과 성능이점이 사라진다.

EnumMap 버전과 Stream 버전은 다르게 동작한다.
Enum 버전은 언제나 식물의 생애주기당 하나씩의 중첩 맵을 만들지만, 스트림 버전에서는 해당 생애주가의 속하는 식물이 있을떄만 만든다.

![](@attachment/Clipboard_2022-04-24-00-55-21.png)



## 심화 EnumMap
다음은 ordinal을 두번이나 쓴 배열들의 배열이다.

두 가지 상태(Phase)를 전이(Transition)와 매핑하는 예제이다. LIQUID에서 SOLID의 전이는 FREEZE가 되고, LIQUID에서 GAS로의 전이는 BOIL이 될것이다. 이번 예제에서도 Phase나 Transition의 상수의 선언 순서를 변경하거나 새로운 Phase 상수를 추가하는 경우에도 문제가 발생할 수 있다.


 ```java

 public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT,FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;

        private static final Transition[][] TRANSITIONS = {
                {null, MELT, SUBLIME},
                {FREEZE, null, BOIL},
                {DEPOSIT, CONDENSE, null}
        };

        public static Transition from(Phase from, Phase to) {
            return TRANSITIONS[from.ordinal()][to.ordinal()];
        }
    }
}

 ```
위 코드를 EnumMap으로 수정

 ```java

import java.util.EnumMap;
import java.util.Map;
import java.util.stream.Collectors;
import java.util.stream.Stream;

enum Phase {

    SOLID, LIQUID, GAS;

    public enum Transition {
        
        MELT(SOLID, LIQUID),
        FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS),
        CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS),
        DEPOSIT(GAS, SOLID);

        private final Phase from;
        private final Phase to;

        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        private static final Map<Phase, Map<Phase, Transition>> transitionMap = Stream.of(values())
                .collect(Collectors.groupingBy(t -> t.from, // 바깥 Map의 Key
                        () -> new EnumMap<>(Phase.class), // 바깥 Map의 구현체
                        Collectors.toMap(t -> t.to, // 바깥 Map의 Value(Map으로), 안쪽 Map의 Key
                                t -> t, // 안쪽 Map의 Value
                                (x,y) -> y, // 만약 Key값이 같은게 있으면 기존것을 사용할지 새로운 것을 사용할지
                                () -> new EnumMap<>(Phase.class)))); // 안쪽 Map의 구현체;

        public static Transition from(Phase from, Phase to) {
            return transitionMap.get(from).get(to);
        }
    }

}
 ```
 너무 복잡한 코드인데 , 
  Map<Phase, Map<Phase, Transition>>은 "이전 상태에서 '이후 상태에서 전이로의 맵'에 대응시키는 맵"이라는 뜻이다.

이 코드에서 PLASMA를 추가하면 전이 목록에 IONIZE(GAS, PLASMA)와 DEIONIZE(PLASMA, GAS)만 추가하면 끝이다.


 ```java
public enum Phase {
    SOLID, LIQUID, GAS, PLASMA;

    public enum Transition {
        MELT(SOLID, LIQUID),
        FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS),
        CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS),
        DEPOSIT(GAS, SOLID),
        IONIZE(GAS, PLASMA),
        DEIONIZE(PLASMA, GAS);
    }
    
    //나머지 코드는 그대로
    
}

 ```
 나머지는 기존 로직에서 잘 처리해주니 잘못 수정할 가능성이 극히 작다. 실제 내부에서는 맵들의 맵이 배뎔들의 배열로 구현되니 낭비되는 공간과 시간도 거의 없이 명확하고 안전하고 유지보수하기 좋다.

# 핵심

>배열의 인덱스를 얻기 위해 ordinal을 쓰는 것이 일반적으로는 좋지 않으니, 대신 EnumMap을 사용해라,
애플리케이션 프로그래머는 Enum.ordinal을 사용하지 말아햐 한다.














