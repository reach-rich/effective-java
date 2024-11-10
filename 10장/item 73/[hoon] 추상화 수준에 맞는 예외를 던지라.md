## 1. 들어가기

개발을 하다보면 어떠한 메서드를 사용할 때 관련 없어 보이는 예외가 나오는 경우가 있는데 그런 경우에는 매우 당황스럽습니다.

보통 해당 메서드가 저수준의 예외를 처리하지 않고 외부로 전파했을 때 종종 일어날 수 있는데 이런 경우 내부 구현 방식을 외부에 드러내어 윗 레벨 API를 오염시킬 가능성이 큽니다.

그러므로 저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 바꿔야합니다.

## 2. 예외 번역

저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 바꾸는 것을 예외 번역이라고 합니다.

그럼 예외 번역을 어떻게 사용할 수 있는지 알아보겠습니다.

```java
   try {
      ...
   } catch (LowerLevelException e) {
      throw new HigherLevelException();
   }
```

해당 코드와 같이 저수준의 예외를 고수준의 예외로 바꿔 외부로 던질 수 있습니다.

대표적인 예로 `AbstractSequentialList`의 `get` 메서드에 구현되어 있습니다.

```java
   public E get(int index) {
      try {
         return listIterator(index).next();
      } catch (NoSuchElementException exc) {
         throw new IndexOutOfBoundsException("Index: "+index);
      }
   }
```

## 3. 예외 연쇄

만약, 디버깅에 도움이 된다면 저수준 예외도 필요할 수 있습니다.

그런 경우에는 저수준의 예외를 고수준 예외에 실어 보낼 수 있는데 이를 예외 연쇄라 합니다.

사용법은 다음과 같습니다.

```java
   try {
      ...
   } catch (LowerLevelException cause) {
      throw new HigherLevelException(cause);
   }
```

그리고 고수준 예외 생성자는 다음과 같이 작성할 수 있습니다.

```java
   class HigherLevelException extends Exception {
      HigherLevelException(Throwable cause) {
         super(cause);
      }
   }
```

## 4. 주의해야 할 점

무턱대고 예외를 전파하는 것보다 예외 번역이 우수한 방법이지만, 남용해서는 안됩니다.

가장 좋은 방법은 저수준 메서드가 반드시 성공하도록 하는 것입니다.

만약, 예외를 피할 수 없다면 내부에서 처리하여 API 호출자에게까지 전파하지 않도록 하는 것이 좋습니다.

그럼 API 호출자에게 예외를 전파하지 않으면서 프로그래머가 로그를 분석해 추가 조치를 취할 수 있습니다.