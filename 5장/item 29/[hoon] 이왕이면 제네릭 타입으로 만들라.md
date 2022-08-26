## 1. 들어가기

JDK가 제공하는 제네릭 타입과 메서드를 사용하는 일은 대체적으로 쉬운 편이지만,

제네릭 타입을 새로 만드는 일은 그에 비해 어렵습니다.

하지만 이를 배워두면 충분히 그만한 값어치를 합니다.

그러므로 이번 포스트는 제네릭 타입을 새로 만드는 방법에 대해 알아보겠습니다.

## 2. 제네릭 타입을 새로 만드는 방법

Item 7에서 다룬 스택 코드를 제네릭 타입으로 새로 만들어 봅시다.

```java
  public class Stack {

    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
      elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
    
    public void push(Object e) {
      elements[size++] = e;
    }

    public Object pop() {
      if(size == 0)
        throw new EmptyStackException();

      Object result = elements[--size];
      elements[size] = null;
      return result;
    }
    ...
  }
```

Item7에서 작성했던 해당 Stack 클래스는 Object 타입을 반환하고 있습니다.

그렇기 때문에 사용할 때마다 형변환을 해야 하는데 이 때 런타임 오류가 발생할 가능성이 있습니다.

이 클래스를 제네릭으로 변경해봅시다.

1. 클래스 선언에 타입 매개변수(E)를 추가한다.

   ```java
    public class Stack<E> {
      private E[] elements;
      private int size = 0;
      private static final int DEFAULT_INITIAL_CAPACITY = 16;

      public Stack() {
        elements = new E[DEFAULT_INITIAL_CAPACITY]; // 컴파일 오류
      }
      
      public void push(E e) {
        elements[size++] = e;
      }

      public E pop() {
        if(size == 0)
          throw new EmptyStackException();

        E result = elements[--size];
        elements[size] = null;
        return result;
      }
      ...
    }
   ```

   하지만 E는 실체화 불가 타입이기 때문에 7번째 줄에서 컴파일 오류가 발생합니다.

   이에 대한 해결법으로는 2가지가 있습니다.

2. 해결법

   * **제네릭 배열 생성 금지 제약을 우회한다.**

      Object 배열을 생성한 다음 제네릭 배열로 형변환 하는 방법입니다.

      ```java
        elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
      ```

      이렇게 작성하게 되면 오류가 경고로 바뀌게 됩니다.

      컴파일러가 타입 안전성을 보장하지 못한다는 말인데요.

      우리는 이 elements 배열의 비검사 형변환이 타입 안전성을 해치지 않음을 증명할 수 있습니다.

      elements는 private이므로 클라이언트로 반환 또는 다른 메서드로 전달되는 일이 전혀 없습니다.

      또한, push 메서드를 통해 배열에 저장되는 원소 타입도 항상 E이므로 타입 안전합니다.

      그러므로 앞서 Item27에서 알아보았던 @SuppressWarnings를 사용해 경고를 제거합시다.

      ```java
        @SuppressWarnings("unchecked")
        public Stack() {
          elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
        }
      ```

    * **elements 필드 타입을 E[]에서 Object[]로 수정한다.**

       배열을 Object 타입으로 변경하고 사용할 때 형변환을 하는 방법입니다.

       ```java
        private Object[] elements;

        public E pop() {
          if(size == 0)
            throw new EmptyStackException();

          @SuppressWarnings("unchecked")
          E result = (E) elements[--size];
          elements[size] = null;
          return result;
        }
       ```

두 방법 모두 나름의 지지를 얻고 있으나 두 방법 모두 장단점이 있습니다.

첫 번째 방법은 가독성이 좋은 반면, 런타임 타입이 컴파일타임 타입과 달라 힙 오염을 일으킵니다.

두 번째 방법은 힙 오염에서 안전하지만, 배열에서 원소를 읽을 때마다 형변환을 해야 합니다.

현업에서는 첫 번째 방식을 더 선호하며 자주 사용하는 편이지만, 두 번째 방법을 사용해도 무방합니다.

## 3. 한정적 타입 매개변수 (Bounded Type Parameter)

타입 매개변수(E)에 제약을 두는 방법도 있는데 이를 **한정적 타입 매개변수**라고 합니다.

그 예로 java.util.concurrent.DelayQueue가 있습니다.

```java
  class DelayQueue<E extends Delayed> implements BlockingQueue<E>
```

이렇게 작성을 하면 java.util.concurrent.Delayed의 하위 타입만 받는다는 뜻이 됩니다.

그럼, 이를 사용하는 클라이언트는 따로 형변환할 필요 없이 바로 사용할 수 있습니다.

추가로, 모든 타입은 Delayed의 하위 타입이므로 다음과 같이 작성할 수도 있습니다.

```java
 class DelayQueue<Delayed> implements BlockingQueue<E>
```

## 4. 정리

이번 포스트에서는 제네릭 타입을 생성하는 방법과 한정적 타입 매개변수에 대해 알아보았습니다.

클라이언트에서 직접 형변환해야 하는 타입보다는 제네릭 타입이 더 안전하고 쓰기 편하기 때문에

이왕이면 **제네릭 타입**으로 설계하고, 기존 타입 중 제네릭이었어야 하는 것은 제네릭 타입으로 변경합시다.

그리고 만약, 타입 매개변수를 제한하고 싶다면 **한정적 타입 매개변수**를 사용해봅시다.