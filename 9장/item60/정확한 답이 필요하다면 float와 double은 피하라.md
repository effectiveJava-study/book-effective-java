# 아이템 60. 정확한 답이 필요하다면 float와 double은 피하라.

넓은 범위의 수를 빠르게 정밀한 ‘근사치'로 계산 하도록 세심하게 설계 되었음.

**float와 double타입은 특히 금융 관련 계산과는 맞지 않는다.**

```java
public static void main(String[] args) {
	double funds = 1.00;
	int itemsBought = 0;
	for (double price = 0.10; funds >= price; price += 0.10) {
		funds -= price;
		itemsBought++;
	}
	System.out.println(itemsBought + "개 구입");
	System.out.println("잔돈(달러):" + funds);
```

결과값은 0.3999999… 가 된다.

### 해결방안

금융 계산에는 BigDecimal(8가지 반올림 모드가 있다고 한다), int 혹은 long을 사용한다.

**하지만** BigDecimal에는 두 가지 단점이 있다.

1. 기본 타입보다 쓰기가 훨씬 불편함
2. 느림

대안으로 int, long을 사용할 수 있지만 다룰 수 있는 값의 크기가 제한되고 소수점을 직접 관리해야한다.
