## 1. 들어가기

숙련된 프로그래머는 그렇지 못한 프로그래머보다 더 많은 코드를 재사용합니다.

예외도 마찬가지로 재사용하는게 좋은데 Java 라이브러리는 API에서 쓰기 충분한 수의 예외를 제공합니다.

이러한 표준 예외를 재사용하면 얻는 장점에 대해 알아보겠습니다.

## 2. 표준 예외 재사용의 장점

* 용이성

   표준 예외를 사용하면 이미 익숙해진 규약을 그대로 따르기 때문에 API를 다른 사람이 사용하기에 용이하다는 장점이 생깁니다.

* 가독성

   낯선 예외를 사용하지 않으므로 읽기 쉬워진다는 장점이 생깁니다.

* 오버헤드 감소

   예외 클래스 수가 적어지기에 메모리 사용량 감소, 클래스 적재 시간 감소 등의 오버헤드가 감소하는 장점이 생깁니다.

## 3. 주로 사용되는 표준 예외

이번에는 API에서 주로 사용되는 표준 예외에 대해 알아보겠습니다.

API에서 사용하는 메서드에 대한 예외를 `IllegalArgumentException`이나 `IllegalStateException`으로 통합하여 사용할 수 있겠지만,
특수한 일부는 따로 구분해서 사용하는 것이 좋습니다.

그 특수한 예외는 아래에 있습니다.

|Exception|쓰임|
|:--------|:---|
|IllegalArgumentException|API 호출자가 API에서 허용하지 않는 값을 인수로 건넸을 때|
|IllegalStateException|객체가 메서드를 수행하기에 적절하지 않은 상태일 때|
|NullPointerException|null을 허용하지 않는 메서드에 null을 건넸을 때|
|IndexOutOfBoundsException|인덱스가 범위를 넘어섰을 때|
|ConcurrentModificationException|허용하지 않는 동시 수정이 발견되었을 때|
|UnSupportedOperationException|호출한 메서드를 지원하지 않을 때|
|ArithmeticException|나누기 0을 했을 때|
|NumberFormatException|문자를 숫자로 변경 불가능할 때|