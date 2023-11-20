---
title: Item 61. 박싱된 기본 타입보다는 기본 타입을 사용하라
categories: [Effective-Java]
tags: [item 61]
toc: true

date: 2023-11-07
last_modified_at: 2023-11-07
---

## 1. 들어가기

자바의 데이터 타입은 크게 두 가지로 나눌 수 있습니다.

바로 int, double과 같은 기본 타입과 String, List와 같은 참조 타입입니다.

각 기본 타입은 오토박싱을 통해 참조 타입을 하나씩 가지는데

먼저, 기본 타입과 박싱된 기본 타입의 차이점에 대해 알아봅시다.

## 2. 기본 타입과 박싱된 기본 타입의 차이점

1. 속성

   기본 타입은 값만 가지고 있으나, 박싱된 기본 타입은 값과 식별성이라는 속성을 가집니다.

   즉, 같은 값을 가져도 서로 다르다고 식별될 수 있다는 의미입니다.

2. null 값

   기본 타입은 항상 유효한 값만 가지나, 박싱된 기본 타입은 유효하지 않은 값 즉, null을 가질 수 있습니다.

3. 효율성

   기본 타입이 박싱된 기본 타입보다 시간과 메모리 사용면에서 더 효율적입니다.

## 3. == 연산자를 사용한 박싱 기본 타입

위에서 알아보았던 세 가지 차이 때문에 주의하지 않고 사용한다면 문제가 발생할 수 있습니다.

예시를 보겠습니다.

```java
   Comparator<Integer> naturalOrder =
      (i, j) -> (i < j) ? -1 : (i == j ? 0 : 1);
```

겉으로 봤을 땐 아무 문제 없어보이는 코드입니다.

하지만 `naturalOrder.compare(new Integer(42), new Integer(42))`의 값을 출력해봅시다.

기대값은 0이지만 실제로는 1을 출력합니다.

그 이유는 naturalOrder의 첫 번째 검사 `i < j`는 잘 이뤄지지만 두 번째 검사 `i == j`에서 문제가 생깁니다.

`i == j`는 객체 참조의 식별성을 검사하는데 42 값을 가진 서로 다른 두 객체이므로 1이 출력되는 것입니다.

이와 같이 박싱된 기본 타입에 아무 생각없이 == 연산자를 사용하면 위험합니다.

다음은 위의 문제를 수정한 예시입니다.

```java
   Comparator<Integer> naturalOrder = (iBoxed, jBoxed) -> {
      int i = iBoxed, j = jBoxed;
      return i < j ? -1 : (i == j ? 0 : 1);
   }
```

## 4. Null 값을 가지는 박싱 기본 타입

다음 예시도 보겠습니다.

```java
   public class Unbelievable {
      static Integer i;

      public static void main(String[] args) {
         if (i == 42)
            System.out.println("믿을 수 없군!");
      }
   }
```

이 예시는 물론 "믿을 수 없군!"을 출력하지 않지만 기이한 결과를 보여줍니다.

`i == 42`를 검사할 때 `NullPointerException`을 던지는 것입니다.

그 이유는 바로, i의 타입이 int가 아닌 Integer라서 초기값이 null이기 때문입니다.

다음은 위의 문제를 수정한 예시입니다.

```java
   public class Unbelievable {
      static int i;

      public static void main(String[] args) {
         if (i == 42)
            System.out.println("믿을 수 없군!");
      }
   }
```

## 5. 성능이 느린 박싱 기본 타입

다음 예시도 보겠습니다.

```java
   public static void main(String[] args) {
      Long sum = 0L;

      for (long i=0; i<=Integer.MAX_VALUE; i++) {
         sum += i;
      }

      System.out.println(sum);
   }
```

이 예시는 정상적으로 합이 출력되나, sum 변수를 박싱 기본 타입으로 사용해 결과가 느리게 출력됩니다.

그 이유는 바로, 박싱과 언박싱이 반복해서 일어나기 때문입니다.

다음은 위의 문제를 수정한 예시입니다.

```java
   public static void main(String[] args) {
      long sum = 0L;

      for (long i=0; i<=Integer.MAX_VALUE; i++) {
         sum += i;
      }

      System.out.println(sum);
   }
```

## 6. 박싱 기본 타입은 언제 써야할까?

위의 3가지 예시에서 박싱 기본 타입의 잘못된 예를 보았습니다.

그럼 언제 박싱 기본 타입을 사용해야 할까요?

1. 타입 매개변수로 사용

   타입 매개변수로 기본 타입을 지원하지 않기 때문에 박싱 기본 타입을 사용해야 합니다.

   `List<Integer>`, `Map<Long, Integer>`

2. 리플렉션

   리플렉션을 통해 메서드를 호출할 때, 기본 타입이 아닌 박싱 기본 타입을 사용합니다.

## 7. 정리

   이번 포스트는 박싱 기본 타입의 잘못된 사용 예와 사용처에 대해 알아보았습니다.

   기본 타입과 박싱된 기본 타입 중 하나를 선택해야 한다면 가능한 간단하고 빠른 기본 타입을 사용합시다.