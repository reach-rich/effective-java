Effective Java의  60번째 아이템 "정확한 답이 필요하다면 float와 double은 피하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 1. float와 double의 문제점

float와 double 타입은 과학과 공학 계산용으로 설계되었다. 이진 부동소수점 연산에 쓰이며, 넓은 범위의 수를 빠르게 정밀한 '근사치'로 계산하도록 설계되었다. 따라서 정확한 결과가 필요할 때는 사용하면 안 된다.

```java
System.out.println(1.03 - 0.42);
// 0.6100000000000001

System.out.println(1.00 - 9 * 0.10);
// 0.0999999999999998
```

반올림하면 해결되리라 생각할 수 있지만 그렇지 않다.

```java
public static void main(String[] args) {
  double funds = 1.00;
  int itemsBought = 0;
  for (double price = 0.10; funds >= price; price += 0.10) {
    funds -= price;
    itemsBought++;
  }
  System.out.println(itemsBought + "개 구입");
  System.out.println("잔돈(달러): " + funds);
}

// 3개 구입
// 잔돈(달러): 0.3999999999999999
```

<br>

## 2. 대안

이 문제를 해결하려면 BigDecimal, int 혹은 long을 사용해야 한다. 아래는 BigDecimal을 사용한 예제다.

```java
public static void main(String[] args) {
  final BigDecimal TEN_CENTS = new BigDecimal(".10");
  
  int itemsBought = 0;
  BigDecimal funds = new BigDecimal("1.00");
  for (BigDecimal price = TEN_CENTS; funds.compareTo(price) >= 0; price = price.add(TEN_CENTS)) {
    funds = funds.subtract(price);
    itemsBought++;
  }
  System.out.println(itemsBought + "개 구입");
  System.out.println("잔돈(달러): " + funds);
}

// 4개 구입
// 잔돈(달러): 0
```

올바른 답이 나왔다. 그러나 BigDecimal은 기본 타입보다 쓰기가 불편하고 느리다. 

<br>

BigDecimal의 대안으로 int 또는 long 타입을 쓸 수도 있다. 그럴 경우 다룰 수 있는 값을 크기가 제한되고, 소수점을 직접 관리해야 한다. 따라서 이번 예에서는 모든 계산을 달러 대신 센트로 수행하면 문제가 해결된다. 

```java
public static void main(String[] args) {
  int itemsBought = 0;
  int funds = 100;
  for (int price = 10; funds >= price; price += 10) {
    funds -= price;
    itemsBought++;
  }
  System.out.println(itemsBought + "개 구입");
  System.out.println("잔돈(달러): " + funds);
}

// 4개 구입
// 잔돈(달러): 0
```

<br>

## 3. 핵심 정리

* 정확한 답이 필요한 계산에는 float와 double은 피하라.
* 소수점 추적은 시스템에 맡기고, 코딩 시의 불편함이나 성능 저하를 신경 쓰지 않겠다면 BigDecimal을 사용하라.
* BigDecimal이 제공하는 여덟 가지 반올림 모드를 이용하여 반올림을 완벽히 제어할 수 있다.
* 성능이 중요하고 소수점을 직접 추적할 수 있고 숫자가 너무 크지 않다면 int나 long을 사용하라.
