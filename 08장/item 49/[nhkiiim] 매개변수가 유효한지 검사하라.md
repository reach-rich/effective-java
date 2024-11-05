# 매개변수가 유효한지 검사하라

메서드와 생성자 대부분은 입력 매개변수의 값이 특정 조건을 만족하기를 바란다.

- 인덱스의 값은 음수면 안됨
- 객체 참조는 null이 아니어야함

> 이런 제약들은 반드시 문서화해야 하며 메서드 몸체가 시작되기 전에 검사해야 잘목된 값이 넘어왔을 때 깔끔한 방식으로 예외를 던질 수 있다.

#
### 매개변수 검사를 제대로 하지 못하면 발생하는 문제

- 메서드가 수행되는 중간에 모호한 예외를 던지며 실패할 가능성 존재
- 메서드가 문제없이 수행됐지만 잘목된 결과를 반환하는 더 나쁜 상황 발생 가능
- 미래의 알 수 없는 시점에 메서드와 관계 없는 오류를 내는 것은 실패 원자성을 어기는 결과를 가져옴

#
### 예외의 문서화
- public, protected 메서드는 매개변수 값이 잘못됐을 때 던지는 예외를 문서화해야 함 (@throws 자바독 태그 사용)
- IllegalArgumentException, IndexOutOfBoundException, NullPointerException 과 같은 예외가 될 것
- 매개변수의 제약을 문서화한다면 그 제약을 어겼을 때 발생하는 예외도 함께 기술해야함

> 전형적인 문서화 예시

```java
/**
* (현재 값 mod m) 값을 반환한다. 이 메서드는 
* 항상 음이 아닌 BigInteger를 반환한다는 점에서 remainder 메서드와 다르다.
*
* @param m 계수 (양수여야 한다.)
* @return 현재 값 mod m
* @throws ArithmeticException m이 0보다 작거나 같으면 발생한다.
*/
public BigInteger mod(BigInteger m) {
    if (m.signum() < 0)
        throw new ArithmeticException("계수(m)는 양수여야 합니다. " + m);
    ...
}
```
- m이 null일 때 호출되는 NullPointerException은 BigInteger 클래스 수준에서 기술
- 클래스 수준의 주석은 그 클래스의 모든 메서드에 적용되기 때문에 깔끔한 방법!
- `@Nullable` 애너테이션을 사용해 null이 될 수 있다 알려주는 방법도 있지만 표준적인 방법은 아님

#
### 예외 처리 방법

> #### requireNotNull
> 자바 7에 추가된 java.util.Objects.requireNotNull 메서드는 유연하고 사용하기 편해 더 이상 null 검사를 수동으로 하지 않아도 된다
> ```java
> this.strategy = Objects.requireNotNull(strategy, "전략");
> ```

<br>

> #### Objects의 범위 검사
> 자바 9에서는 Objects에 범위 검사 기능도 더해졌다
> - `checkFromIndexSize`, `checkFromToIndex`, `checkIndex` 메서드 존재
> - 예외 메시지를 지정할 수 는 없고 리스트와 배열 전용으로 설계
> - 닫힌 범위(양 끝단)를 다루지 못함 주의

> #### 단언문
> public이 아닌 공개되지 않은 메서드는 호출되는 상황을 통제해야함
> - 단언문(assert)을 사용해 매개변수 유효성을 검증
> - 단언문은 일반적인 유효성 검사와 다르게 실패 시 AssertionError를 던짐
> - 런타임에 아무런 효과도, 성능 저하도 없음
> 
> ```java
> private static void sort(long a[], int offset, int length) {
>   assert a ! = null;
>   assert object >= 0 && offset <= a.length;
>   assert length >= 0 && length <= a.length - offset;
>   ...
> ```

#
### 주의 사항
- 메서드가 직접 사용하지는 않으나 나중에 쓰기 위해 저장하는 매개변수는 특히 신경써서 검사 필요
- 메서드 몸체 실행 전에 매개변수 유효성을 검사해야한다는 규칙에도 예외 존재
  - 유효성 검사 비용이 지나치게 높거나 실용적이지 않을 때  
  - 계산 과정에서 암묵적으로 검사가 수행될 때 (정렬하는 메서드에서는 비교 과정에서 암묵적으로 검사 수행)
- 계산 과정에서 필요한 유효성 검사가 이뤄지지만 잘못된 예외를 던지기도 함 -> 예외 번역 관용구를 사용해 API 문서에 기재된 예외로 번역 필요

> 이번 내용이 무조건 매개변수에 제약을 두라는 의미는 아니다~!  
> 매개변수는 최대한 범용적으로 설계해야하며 제약이 적을수록 좋다!


