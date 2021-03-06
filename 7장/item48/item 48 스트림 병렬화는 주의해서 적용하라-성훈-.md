# item 48 스트림 병렬화는 주의해서 적용하라

자바는 항상 동시성 프로그래밍 측면에서 앞서갔다.

자바 5 부터 동시성 컬렉션인 java.util.concurrent 라이브러리와 실행자(Executor) 프레임워크를 지원했다.

자바 7 부터는 고성능 병렬 분해(parallel decom-position) 프레임워크인 포크-조인(fork-join) 패키지를 추가.

그리고 자바 8부터는 parallel 메서드만 한 번 호출하면 파이프라인을 병렬 실행할 수 있는 스트림을 지원했다.

이처럼 자바로 동시성 프로그램 작성하기가 점점 쉬워지고 있지만, 이를 올바르고 빠르게 작성하는 일은, 여전히 어려운 작업이다.

동시성 프로그래밍을 할 때는 **`안전성(safety)`** 과 **`응답가능(liveness)`** 상태를 유지해야 한다.

## 동시성(Concurrency) vs 병렬성(Parallelism)
![image](https://user-images.githubusercontent.com/90130141/168432048-ff2062b5-b113-44c9-a567-952ccaf5d3d5.png)

-------------`순차적인`-----------------      `동시` ---------------   `병렬`------------
![image](https://user-images.githubusercontent.com/90130141/168431985-80e52e12-20a2-4bfe-a981-905bde869976.png)


병렬성도 동시성을 의미하지만 병렬성과 동시성과의 차이는 `각 코어내의 스레드가 실제로 동시에 명령어를 실행할 수 있음을 말한다`. 그러므로 두개의 알고리즘이 정확히 같은 시점에 실행될 때 이를 병렬적이라고 말할 수 있다.

---
## 스트림 병렬화의 문제점

```java

    public static void main(String[] args){
        primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
                .filter(mersenne -> mersenne.isProbablePrime(50))
                .limit(20)
                .forEach(System.out::println);
    }
    static Stream<BigInteger> primes(){
        return Stream.iterate(TWO, BigInteger::nextProbablePrime);
    }

```
![](@attachment/Clipboard_2022-05-14-22-11-24.png)
이 프로그램을 실행하면 즉각 소수를 찍기 시작해  10초만에 완료된다

여기서 속도를 높이고 싶다고 스트림 파이프라인의 paralle()을 호출하면 하겠다고 한다면 어떻게 될까.

안타깝게도 아무것도 출력하지 못하면서 CPU는 90% 잡아먹는 상태가 무한이 지속된다 이유는 스트림 라이브러리가 이 파이프라인을 병렬화 하는 방법을 찾아내지 못했기 때문이다.

환경이 아무리 좋아도 **데이터 소스가 Stream.iterate거나 중간 연사으로 limit을 쓰면 파이프라인 병렬화로는 성능 개선을 기대할 수 없기 때문이다**

>스트림 파이프라인을 마구잡이로 병렬화 하면 안된다

---

## 병렬화 효과가 좋은 스트림 소스
`ArrayList`, `HashMap`, `HashSet`, `ConcurrentHashMap`의 인스턴스거나 `배열`, `int 범위`, `long 범위`일 때 

이 자료구조들은 모두 데이터를 원하는 크기로 정확하고 손쉽게 나눌 수 있기 대문에 일을 다수의 스레드로 분배하기 좋다는 특징이 있음

또한 원소들을 순차적으로 실행할 때의 참조 지역성(locality of reference) 가 뛰어나다.
>참조 지역성:
코드나 데이터 등이 짧은시간내에 재사용되는 프로그램의 특성



---

## 스트림 병렬화를 적용하기 전에 고려 사항
* 스트림 병렬화는 오직 성능 최적화 수단이다.

* 성능 테스트를 통해 가치가 있는지 확인해야 함

* 보통은 병렬 스트림 파이프라인도 공통의 포크-조인 풀에서 수행(같은 스레드 풀 사용)되므로 잘못된 파이프라인 하나가 다른 부분의 성능에까지 악영향을 미칠수 있다.

---




# 핵심

>* 계산도 올바로 수행하고 성능도 빨라질 거라는 확신 없이는 스트림 파이프라인 병렬화는 시도조차 하지 말라.
>
>* 스트림을 잘못 병렬화하면 프로그램을 오동작하게 하거나 성능을 급격히 떨어뜨린다.
>
> * 계산도 정확하고 성능도 좋아졌음이 확실해졌을 때만 스트림 병렬화를 사용하라.
