## item11. equals를 재정의하려거든 hashCode도 재정의하라 

* **`equals :`** 두 객체의 내용이 같은지, 동등성(equality)를 비교하는 메서드

* **`hashCode :`** 두 객체가 같은 객체인지, 동일성(identity)를 비교하는 메서드



---



### equals 를 재정의한 클래스 모두에서 hashCode도 재정의해야 한다.


**`<hashCode의 3가지 규약>`**
  
 equals를 재정의 한 후 hashCode도 재정의 하지 않으면 hashCode 일반 규약을 어기게 되는 것이다.



* equals가 변경되지 않았다면 애플리케이션이 실행되는 동안 그 객체의 hashCode는 항상 같은 값을 반환해야한다. 단, 애플리케이션이 재실행하는 경우 값이 달라질 수는 있다.

* equals가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야한다.

* equals가 두 객체를 다르다고 판단하더라도 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시 테이블의 성능이 좋아진다.



**hashCode 재정의를 잘못했을 때 크게 문제가 되는 조항은 두 번째다.**
즉, equals를 재정의하여 논리적으로 같다고 정의했을때 만약 hashCode를 기본 구현으로 사용한다면 서로 다른 값을 반환한다. 즉, equals 재정의로 2번 규약을 어기게되는 것이다.


특히 hashCode는 hash를 기반으로 하는 컬렉션 HashMap과 HashSet 등에 매우 중요하다.


**hashCode()**

*  객체의 hashCode를 리턴하는 메서드

* `hashCode` : 객체의 주소값을 변환하여 생성한 객체의 고유한 정수값이다.

간단한 테스트 코드 예제
```
class MemberTest {
    @DisplayName("hashCode 에 대해 알아보기")
    @Test
    void hoon의_hashCode(){
        Member hoon = new Member("hoon", "man", "24");
        Member hoon1 = new Member("hoon1", "man", "24");
        Member hoon2 = hoon; // 같은 객체
        System.out.println(hoon.getName() + "의 hashCode" + hoon.hashCode());
        System.out.println(hoon1.getName() + "의 hashCode" + hoon1.hashCode());
        System.out.println(hoon.hashCode()+" "+hoon2.hashCode()+" 비교");
        assertEquals(hoon.hashCode(), hoon2.hashCode());
    }
}
```
>hoon와 hoon2는 논리적으로 같은 객체(동일한 주소값을 가진다) 이므로 hashCode 또한 같다.

### 좋은 hashCode를 작성하는 간단한 요령

**나쁜거**
```
@Override 
public int hashCode() {
     return 42; 
}
```
모든 객체가 해시테이블의 버킷 하나에 담겨 LinkedList 처럼 동작하게 된다. → 평균 수행 시간이 O(n)으로 느려져서 객체가 많아지면 쓸 수 없게 된다.

* 좋은 해시 함수라면 서로 다른 인스턴스에 다른 해시코드를 반환해야 한다. 

**좋은 거**

1. int 변수 reult 를 선언 한 후 값 c로 초기화 한다. 이때 c는 해당 객체의 첫번째 핵심 필드를 (2.a)단계 방식으로 계산한 hashCode다.

2. 해당 객체의 나머지 핵심필드 f 각각에 대해 다음 작업을 수행한다.

a. 해당 필드의 해시코드 c를 계산한다.

i. 기본 타입 필드라면, Type.hashCode(f)를 수행한다. (Type: 해당 기본 타입의 박싱 클래스)

ii. 참조 타입 필드면서 이 클래스의 equals 메서드가 이 필드의 equals를 재귀적으로 호출해 비교한다면, 이 필드의 hashCode 를 재귀적으로 호출한다. 계산이 더 복잡해질 것 같으면, 이 필드의 표준형을 만들어 그 표준형의 hashCode를 호출한다. 필드의 값이 null이면 0을 사용한다. (보통 0을 사용)

iii. 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다. 이상의 규칙을 재귀적으로 적용해 각 핵심 원소의 해시코드를 계산한 다음, (2.b)단계 방식으로 갱신한다. 배열에 핵심 원소가 하나도 없다면 단순히 상수(0을 추천)를 사용한다. 모든 원소가 핵심 원소라면 Arrays.hashCode를 사용한다.

b. (2.a) 단계에서 계산한 해시코드 c 로 result 를 갱신한다.

result = 31 * result + c;
**31 을 곱하는 이유 : 31은 홀수이며 소수이기 때문**

**주의**

* 파생 필드는 해시코드 계산에서 제외해도 된다.
* eqauls 비교에 사용되지 않은 필드는 반드시 제외해야 한다. (두번째 규약내용)
* hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말자.
* 클라이언트에서 이 값에 의존하지 않고 추후에 계산 방식을 바꿀 수 있게 한다.

>**핵심**
* 성능을 높인답시고 해시코드를 계산할 때 핵심 필드를 생략해서는 안 된다. 해시 품질이 나빠져 해시테이블의 성능을 떨어 트릴수 있다.
>
>
* hashMap, hashSet과 같은 컬렉션의 원소를 사용할때 문제를 일으킬수 있다.
>
>
* 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.
