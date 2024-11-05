## 1. 들어가기

개발을 하다보면 런타임 오류가 발생하는 일이 종종 있습니다.

보통 런타임 오류는 입력 매개변수가 특정 조건을 만족하지 않아 발생하기 쉬운데

예를 들면 null 객체의 속성을 참조하려 할 때나 또는 값을 음수로 나눌 때 발생할 수 있습니다.

그래서 오류는 가능한 한 빨리(발생한 곳에서) 잡아야 합니다.

오류를 발생한 즉시 잡지 못하면 해당 오류를 감지하기 어려워지고, 오류 발생 지점을 찾기 어려워집니다.

그럼 오류를 어떻게 조기에 감지할 수 있을까요?

## 2. 오류 조기 감지 방법

1. public과 protected 메서드는 매개변수 값이 잘못됐을 때 던지는 예외를 문서화해야 한다.

   예외를 문서화할 때는 보통 자바독 태그의 `@throws`를 사용합니다.

   ```java
      /**
       * (현재 값 mod m) 값을 반환
       * 항상 음이 아닌 BigInteger를 반환한다는 점에서 remainder와 다름
       *
       * @param m 계수(양수)
       * @return 현재 값 mod m
       * @throws ArithmeticException m이 0 이하이면 발생
       */
      public BigInteger mod(BigInteger m) {
         if (m.signum() <= 0) {
            throw new ArithmeticException("계수(m)는 양수여야 합니다. " + m);
         }
         return m;
      }
   ```

   추가로 주의깊게 봐야할 곳은 m이 null이면 `m.signum()` 호출 때 NPE를 던진다는 것입니다.

   그럼에도 그에 대한 예외 설명을 하지 않은 이유는 BigInteger 클래스 수준에서 기술했기 때문입니다.

   클래스 수준 주석은 그 클래스의 모든 public 메서드에 적용되므로

   각 메서드에 일일이 기술하는 것보다 훨씬 깔끔한 방법입니다.

   <br>

2. Java 7에서 추가된 `requiredNonNull`

   `requiredNonNull` 메서드는 null 검사를 수동으로 하지 않아도 된다는 장점이 있습니다.

   값이 null일 경우, NPE가 발생함과 동시에 원하는 예외 메시지를 지정할 수 있을 뿐만 아니라

   입력을 그대로 반환하므로 값을 사용함과 동시에 null 검사를 수행할 수 있습니다.

   ```java
      this.strategy = Objects.requireNonNull(strategy, "전략 값이 null입니다.");
   ```

   <br>

3. Java 9에서 추가된 범위 검사 기능

   Java 9에서는 Objects에 범위 검사 기능도 더해졌습니다.

   `checkFromIndexSize`, `checkFromToIndex`, `checkIndex` 라는 메서드들인데
   
   예외 메시지 지정이 불가능하고 리스트, 배열 전용이며 닫힌 범위(양 끝단 포함)는 다루지 못해
   
   `requiredNonNull` 보다 유연하지 않지만 이런 제약이 문제가 안된다면 유용하게 사용할 수 있습니다.

   <br>

4. private 메서드에서의 `assert`

   공개되지 않은 메서드라면 단언문(assert)을 통해 직접 메서드 호출 상황을 통제할 수 있습니다.

   단언문들은 자신이 단언한 조건이 무조건 참이어야 한다는 특징이 있습니다.

   ```java
      private static void sort(long a[], int offset, int length) {
         assert a != null;
         assert offset >= 0 && offset <= a.length;
         assert length >= 0 && length <= a.length - offset;
      }
   ```

   이 단언문들은 아래와 같이 몇 가지 면에서 일반적인 유효성 검사와 다릅니다.

   1. 실패하면 AssertionError를 던진다.

   2. 런타임에 아무런 효과도, 아무런 성능 저하도 없다.

## 3. 주의사항

1. 메서드가 직접 사용하지는 않으나 나중에 쓰기 위해 저장하는 매개변수는 신경써서 검사해야 한다.

2. 유효성 검사 비용이 지나치게 높거나 실용적이지 않을 때는 검사하지 않는다.

3. API 문서에서 작성한 예외와 다른 경우에는 예외 번역 관용구를 사용한다.

## 4. 정리

이번 포스트는 매개변수의 유효성 검사에 대해 알아보았습니다.

항상 메서드나 생성자를 작성할 때는 그 매개변수에 어떤 제약이 있을지 생각해서

그 제약들을 문서화하고 메서드 코드 시작 부분에서 명시적으로 검사해야 합니다.

그리고 그 결과는 실제 오류를 첫 부분에서 걸러냈을 때 효과를 발휘할 것입니다.