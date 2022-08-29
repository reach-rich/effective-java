Effective Java의 스물아홉 번째 아이템 "이왕이면 제네릭 타입으로 만들라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 1. 제네릭 타입 만들기

Item 7에서 다룬 단순한 스택 코드를 다시 살펴보자.

```java
public class Stack {
  private Object[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;

  public Stack() {
    elements = new Object[DEFAULT_INITIAL_CAPACITY];
  }

  public void push(Object e) {
    ensureCapacity();
    elements[size++] = e;
  }

  public Object pop() {
    if (size == 0)
      throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null;
    return result;
  }
  
  public boolean isEmpty() {
    return size == 0;
  }

  private void ensureCapacity() {
    if (elements.length == size)
      elements = Arrays.copyOf(elements, 2 * size + 1);
  }
}
```

이 클래스는 원래 제네릭 타입이어야 마땅하다. 지금 상태에서의 클라이언트는 스택에서 꺼낸 객체를 형변환해야 하는데, 이때 런타임 오류가 날 위험이 있다. 그러니 제네릭 타입으로 만들어보자.

<br>

### 1) 클래스 선언에 타입 매개변수 추가한다.

스택이 담을 원소의 타입을 추가한다. 타입 이름으로는 보통 E를 사용한다.



### 2) 코드에 쓰인 Object를 적절한 타입 매개변수로 바꾼다.

```java
public class Stack<E> {
  private E[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;

  public Stack() {
    elements = new E[DEFAULT_INITIAL_CAPACITY];
  }

  public void push(E e) {
    ensureCapacity();
    elements[size++] = e;
  }

  public E pop() {
    if (size == 0)
      throw new EmptyStackException();
    E result = elements[--size];
    elements[size] = null;
    return result;
  }
  
  ...
}
```

이 단계에서 대체로 하나 이상의 오류나 경고가 뜨는데, 이 클래스도 예외는 아니다. 

`error: elements = new E[DEFAULT_INITIAL_CAPACITY];`

E와 같은 실체화 불가 타입으로는 배열을 만들 수 없다.

<br>

해결 방법은 두 가지다. 

**첫 번째는 제네릭 배열 생성을 금지하는 제약을 대놓고 우회하는 방법이다.** 

Object 배열을 생성한 다음 제네릭 배열로 형변환하자. 이제 컴파일러는 오류 대신 경고를 내보낼 것이다.

`warning: elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];`

이제 이 비검사 형변환이 프로그램의 타입 안전성을 해치지 않음을 우리 스스로 확인한다. 문제의 배열 elements는 private 필드에 저장되고, 클라이언트로 반환되거나 다른 메서드에 전달되는 일이 전혀 없다. 따라서 이 비검사 형변환은 확실히 안전하다.

비검사 형변환이 안전함을 직접 증명했다면 범위를 최소로 좁혀 @SuppressWarnings 어노테이션으로 해당 경고를 숨긴다. 이제 Stack은 깔끔히 컴파일되고, 명시적으로 형변환하지 않아도 ClassException 걱정 없이 사용할 수 있게 된다. 

```java
@SuppressWarnings("unchecked")
public Stack() {
  elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
```

<br>

**두 번째는 elements 필드의 타입을 E[]에서 Object[]로 바꾸는 것이다.**

이렇게 하면 첫 번째와는 다른 오류가 발생한다. 

`error: E result = elements[--size];`

배열이 반환한 원소를 E로 형변환하면 오류 대신 경고가 뜬다.

`warning: E result = (E) elements[--size];`

E는 실체화 불가 타입으므로 컴파일러는 런타임에 이뤄지는 형변환이 안전한지 증명할 방법이 없다. 이번에도 마찬가디로 우리가 직접 증명하고 경고를 숨길 수 있다.

```java
public E pop() {
  if (size == 0)
    throw new EmptyStackException();
  
  @SuppressWarnings("unchecked") E result = (E) elements[--size];
  
  elements[size] = null;
  return result;
}
```

<br>

위의 제네릭 배열 생성을 제거하는 두 방법 모두 나름의 지지를 얻고 있다. 

첫 번재 방법은 가독성이 좋으며 배열의 타입을 E[]로 선언하여 오직 E 타입 인스턴스만 받음을 확실히 어필하고, 형변환을 배열 생성 시 단 한번만 해주면 된다. 하지만 배열의 런타임 타입이 컴파일타임 타입과 달라 힙 오염을 일으킬 수 있다. 따라서 힙 오염이 맘에 걸리는 프로그래머는 두 번째 방식을 고수한다.

<br>

## 2. 핵심 정리

- 클라이언트에서 직접 형변환해야 하는 타입보다 제네릭 타입이 더 안전하고 쓰기 편하다. 그러니 새로운 타입을 설계할 때는 형변환 없이도 사용할 수 있도록 하라.
- 그렇게 하려면 제네릭 타입으로 만들어야 할 경우가 많다. 기존 타입 중 제네릭이었어야 하는 것이 있다면 제네릭 타입으로 변경하자.

<br>

## 3. Related Posts

- E 타입 (Item 68)
- [@SuppressWarnings (Item 27)](https://heung27.github.io/posts/item-27-%EB%B9%84%EA%B2%80%EC%82%AC-%EA%B2%BD%EA%B3%A0%EB%A5%BC-%EC%A0%9C%EA%B1%B0%ED%95%98%EB%9D%BC/)
- 힙 오염 (Item 32)
