## 1. 들어가기

메서드가 던지는 예외는 그 메서드를 올바르게 사용하는데 아주 중요한 정보입니다.

이번에는 메서드가 던지는 예외를 각 예외 별로 문서화하는 방법에 대해 알아보겠습니다.

## 2. 검사 예외의 문서화 방법

검사 예외는 공통 상위 클래스 (ex. `Exception`)로 선언하지 않고 따로 선언하며 각 예외가 발생하는 상황을 @throws 태그를 사용하여 정확히 문서화해야 합니다.

만약 공통 상위 클래스로 선언한다면 메서드 사용자에게 각 예외를 대처할 수 있는 힌트를 주지 못하고, 같은 맥락에서 발생할 여지가 있는 다른 예외들까지 삼켜버릴 수 있어
API 사용성을 크게 떨어뜨리기 때문입니다.

하지만 이 규칙에 유일한 예외도 있는데 그것은 바로 JVM만 호출할 수 있는 main 메서드입니다.

## 3. 비검사 예외의 문서화 방법

일반적인 프로그래밍 오류를 뜻하는 비검사 예외도 검사 예외처럼 문서화해두면 좋습니다.

자신이 일으킬 수 있는 오류들이 무엇인지 알려주면 프로그래머는 자연스럽게 해당 오류가 나지 않도록 코딩하기 때문입니다.

특히, 비검사 예외를 문서로 남기는 것은 인터페이스 메서드에서 특히 중요한데 
이 조건이 인터페이스의 일반 규약에 속하게 되어 그 인터페이스를 구현한 모든 구현체가 일관되게 동작하도록 해주기 때문입니다.

단, 비검사 예외는 메서드 선언의 @throws 목록에는 넣지 않습니다.

검사, 비검사 예외에 따라 API 호출자가 해야 할 일이 달라지므로 구분하는게 좋은데 
자바독 유틸리티는 throws 절에 등장하고 메서드 주석의 @throws 태그에도 명시한 예외와 @throws 태그에만 명시한 예외를 시각적으로 구분함으로써 검사, 비검사 예외를 구분할 수 있습니다.

## 4. 비검사 예외의 문서화가 불가능한 경우

비검사 예외는 웬만하면 문서화하는 것이 좋으나 현실적으로 불가능한 경우도 있습니다.

예를 들어 다른 프로그래머가 작성한 클래스를 사용하는 경우, 
후에 이 클래스가 수정되어 새로운 비검사 예외를 던진다면 이 클래스를 사용하는 메서드는 문서에 언급되지 않은 새로운 비검사 예외를 던질 것입니다.

```java
/* 외부 클래스 */
  public class OuterClass {
    /**
     * @throws NullPointerException
     * @throws IndexOutofBoundsException  <-- 추가
     */
    public static void outerMethod() {
      ...
    }
  }

/* 외부 클래스를 사용하는 클라이언트 */
  public class Client {

    /**
     * @throws NullPointerException
     */
    public void usage() {
      ...
      OuterClass.outerMethod(); // <-- IndexOutofBoundsException 발생 가능성
    }
  }
```

## 5. 중복 예외를 문서화하는 경우

클래스에 많은 메서드가 존재하고 그 메서드들이 같은 예외를 던지는 경우에 각 메서드마다 동일한 예외를 문서화하기는 매우 피곤한 작업입니다.

그러므로 같은 예외인 경우에는 클래스 설명에 추가하는 방법을 사용할 수 있습니다.

```java
/* 계산 클래스 (수정 전) */  
  public class Calculator {
    /**
     * @throws CalculatorException
     */
    public void add() throws CalculatorException  { ... }

    /**
     * @throws CalculatorException
     */
    public void minus() throws CalculatorException { ... }
  }
```

```java
/* 계산 클래스 (수정 후) */  
  /**
   * @throws CalculatorException
   */
  public class Calculator {
    public void add() throws CalculatorException  { ... }
    public void minus() throws CalculatorException { ... }
  }
```