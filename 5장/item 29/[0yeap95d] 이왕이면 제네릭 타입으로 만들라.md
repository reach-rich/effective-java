

### ❓ 왜 제네릭 타입으로 만들어야하는가

JDK가 제공하는 제네릭 타입과 메서드를 사용하는 일은 일반적으로 쉬운 편이 아니지만, 제네릭 타입을 만드는 일은 조금 더 어렵다

하지만 배워두면 그만한 값어치는 충분히 한다

<br>

**✏ #01 예제소스 | Object 기반 스택 - 제네릭이 절실**

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
        if (size == 0) throw new EmptyStackException();
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

> 스택에서 꺼낸 객체 형변환 과정에 런타임 오류 발생 가능성이 존재
>
> 이 클래스는 원래 제네릭 타입이어야 마땅하다

<br>

---

<br>

### ⚙ 일반 클래스를 제네릭 클래스로 만들기

**1️⃣ 클래스 선언에 타입 매개변수를 추가**

- 타입 이름은 보통 `E`를 사용

**✏ #02 예제소스**

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
        if (size == 0) throw new EmptyStackException();
        E result = elements[--size];
        elements[size] = null;
        return result;
    }
    ...
}
```

<br>

**2️⃣ 배열을 사용하는 코드를 제네릭으로 만들려 할 때 발생하는 오류 해결**

```tex
Stack.java:8: generic array creation
    elements = new E[DEFAULT_INITIAL_CAPACITY];
				   ^
```

1. 제네릭 배열 생성을 금지하는 제약을 대놓고 우회하는 방법

   >- 오류 대신 경고로 바뀜
   >- 일반적으로 타입 안전하지 않음
   >- 비검사 형변환이 안전함을 직접 증명하고 `@Suppress Warning` 애너테이션으로 경고 숨김

2. `elements`필드의 타입을 `E[]`에서 `Object[]`로 바꾸는 방법

   > - 또 다른 오류가 발생
   >
   > - 배열이 반환한 원소를 `E`로 형변환하면 오류에서 경고로 바뀜
   >
   > - 직접 증명 후 경고를 숨길 수 있다 (`pop`메서드 전체에서 경고를 숨기지말고, 비검사 형변환을 수행하는 할당문만)
   >
   >   ```java
   >   public E pop() {
   >       if (size == 0) throw new EmptyStackException();
   >   
   >       @SuppressWarnings("unchecked") 
   >       E result = elements[--size];
   >   
   >       elements[size] = null;
   >       return result;
   >   }
   >   ```

***1번 방식이 가독성도 좋고, 코드도 짧고, 배열 생성 시 형변환이 한 번만 필요해서 더 선호하지만 힙 오염을 일으키기 때문에 두 번째 방식을 고수하기도 한다***

<br>

---

<br>

### 📖 제네릭 타입 매개변수 제약

**대다수의 제네릭 타입은 타입 매개변수에 아무런 제약을 두지 않는다** 

- `Stack<Object>`, `Stack<int[]>`, `Stack<List<String>>`, `Stack` 등 어떤 참조 타입으로도 `Stack`을 만들 수 있다

- 단, `Stack<int>`나 `Stack<double>`같은 기본 타입은 사용할 수 없다

- 자바 제네릭 타입 시스템의 근본적인 문제이지만, 박싱된 기본 타입을 사용해 우회할 수 있다

<br>

**매개변수에 제약을 두는 제네릭 타입도 존재한다**

```java
class DelayQueue<E extends Delayed> implements BlockingQueue<E>
```

- `<E extends Delayed>`는 `java.util.concurrent.Delayed`의 하위 타입만 받는다는 뜻

- `DelayQueue` 자신과 `DelayQueue`를 사용하는 클라이언트는 `DelayQueue`의 원소에서 형변환 없이 곧바로 `Delayed` 클래스의 메서드 호출 가능

- 모든 타입은 자기 자신의 하위타입이므로 `DelayQueue<Delayed>`로도 사용할 수 있음

<br>

---

<br>

### 📌 핵심정리

**클라이언트에서 직접 형변환해야 하는 타입보다 제네릭 타입이 더 안전하고 쓰기 편하다**

**그러니 새로운 타입을 설계할 때는 형변환 없이도 사용할 수 있도록 하라**

**그렇게 하려면 제네릭 타입으로 만들어야 할 경우가 많다**

**기존 타입 중 제네릭이었어야 하는 게 있다면 제네릭 타입으로 변경하자**

**기존 클라이언트에는 아무 영향을 주지 않으면서, 새로운 사용자를 훨씬 편하게 해주는 길이다**

<br>
