## 1. 들어가기

대부분 소수점을 표현하고자 할때는 float 또는 double을 사용합니다.

float와 double은 넓은 범위의 수를 빠르고 정밀하게 **"근사치"**로 계산하도록 설계되어 있습니다.

만약, 정확한 결과가 필요할 때 float와 double을 사용한다면 어떻게 될까요?

## 2. float와 double을 잘못 사용한 예

주머니에는 1달러가 있고, 선반에는 10센트, 20센트 ... 1달러짜리의 사탕이 놓여있다고 가정해봅시다.

10센트 사탕부터 차례대로 구입한다고 가정했을 때 아래와 같은 코드로 구현할 수 있습니다.

```java
   public static void main(String[] args) {
      double funds = 1.00;
      int itemsBought = 0;
      for (double price = 0.10; funds >= price; price += 0.10) {
         funds -= price;
         itemsBought++;
      }

      System.out.println(itemBought + "개 구입");
      System.out.println("잔돈(달러): " + funds);
   }
```

위의 예시에서 사탕을 3개 구입한 후에 예상 잔액은 0.4달러입니다.

하지만 실제 잔액은 0.3999999999999달러로 잘못된 결과가 나타납니다.

그럼 어떻게 이 문제를 해결할 수 있을까요?

## 3. 소수점 해결법

앞서 발생한 문제는 BigDecimal, int 혹은 long으로 해결할 수 있습니다.

우선, BigDecimal을 이용한다면 다음과 같이 구현할 수 있습니다.

```java
   public static void main(String[] args) {
      final BigDecimal TEN_CENTS = new BigDecimal(".10");

      int itemsBought = 0;
      BigDecimal funds = new BigDecimal("1.00");
      for (BigDecimal price = TEN_CENTS; funds.compareTo(price) >= 0; price = price.add(TEN_CENTS)) {
         funds = funds.subtract(price);
         itemBought++;
      }

      System.out.println(itemBought + "개 구입");
      System.out.println("잔돈(달러): " + funds);
   }
```

이 프로그램을 실행하면 사탕 4개를 구입 후, 잔돈은 0달러가 남습니다.

올바른 답이 나왔지만 BigDecimal은 성능이 좋지 않고 쓰기 불편하다는 단점이 있습니다.

BigDecimal의 대안으로 숫자가 너무 크지 않다면 int나 long을 통해 관리할 수도 있습니다.

다음은 int와 long으로 구현한 예시입니다.

```java
   public static void main(String[] args) {
      int itemsBought = 0;
      int funds = 100;
      for (int price = 10; funds >= price; price += 10) {
         funds -= price;
         itemsBought++;
      }

      System.out.println(itemBought + "개 구입");
      System.out.println("잔돈(달러): " + funds);
   }
```

## 4. 정리

이번 포스트는 우리가 많이 사용하는 float와 double에 대해 알아보았습니다.

float와 double은 근사치를 표현하므로 정확한 결과를 원하는 경우에 올바른 결과를 도출하지 않습니다.

정확한 결과를 원하는 경우에는 BigDecimal을 사용해봅시다.

BigDecimal은 여덟 가지 반올림 모드를 이용하여 비즈니스 계산에서 아주 편리하게 사용할 수 있습니다.

하지만, 성능이 중요하고 숫자가 너무 크지 않다면 int나 long을 고려해봅시다.