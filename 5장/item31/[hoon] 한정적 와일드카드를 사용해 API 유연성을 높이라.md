## 1. 들어가기

매개변수화 타입은 불공변입니다.

즉, 다른 타입 Type1과 Type2가 있을 때, List\<Type1>와 List\<Type2>는 상하 관계가 성립되지 않습니다.

이런 경우에 어떤 문제가 있을 수 있을까요?

## 2. 일반 매개변수 타입의 문제점

public API인 Stack 클래스를 예시로 사용하겠습니다.

```java
  public class Stack<E> {
    public Stack();
    public void push(E e);
    public E pop();
    public booelan isEmpty();
  }
```

Stack에 일련의 원소를 넣는 메서드를 추가해야한다고 가정하면 다음과 같이 작성할 수 있습니다.

```java
  public void pushAll(Iterable<E> src) {
    for(E e : src)
      push(e);
  }
```

하지만 여기에는 문제가 발생합니다.

예를 들어 Stack\<Number>로 선언한 후 Integer 타입의 값을 pushAll의 인자로 넣으면 어떻게 될까요?

타입 매개변수화 타입이 아닌 일반 구문에서는 `Number num = intValue;`가 성립하지만,

타입 매개변수화 타입은 Stack\<Number>와 Stack\<Integer>는 다른 타입이기 때문에 성립되지 않습니다.

Stack에 있는 원소를 모두 빼서 새로운 컬렉션에 옮겨 담는 메서드도 마찬가지로 문제가 발생합니다.

```java
  public void popAll(Collection<E> dst) {
    while(!isEmpty())
      dst.add(pop());
  }
```

그럼 이것을 어떻게 해결할 수 있을까요?

## 3. PECS 공식

위의 문제점은 PECS 공식과 와일드카드 타입으로 해결할 수 있습니다.

PECS 공식이란, `Producer-Extends, Consumer-Super` 라는 의미로

매개변수화 타입 T가 생산자라면 `<? extends T>`를 사용하고, 소비자라면 `<? super T>`를 사용합니다.

그럼 PECS 공식을 사용해 위의 문제점을 해결하면

pushAll은 Stack이 사용할 E 인스턴스를 생산하므로 `? extends E`를 사용하면 됩니다.

```java
  public class Stack<? extends E> {
    public Stack();
    public void push(E e);
    public E pop();
    public booelan isEmpty();
  }
```

한편, popAll의 dst 매개변수는 Stack으로부터 E 인스턴스를 소비하므로 `? super E`를 사용하면 됩니다.

```java
  public void popAll(Collection<? super E> dst) {
    while(!isEmpty())
      dst.add(pop());
  }
```

## 4. PECS 공식 활용

이번에는 PECS 공식을 활용해 기존에 사용했던 메서드를 수정해보겠습니다.

1. 제네릭 메서드

   ```java
    public static <E> Set<E> union(Set<E> s1, Set<E> s2)
   ```

   위의 예시는 PECS 공식에 따라 생성자인 s1, s2를 `? extends E`로 수정해야 합니다.

   ```java
    public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2)
   ```

   하지만, 반환 타입에도 와일드카드 타입을 적용한다면 
   
   다음과 같이 클라이언트 코드에서 와일드카드 타입을 써야하므로 반환 타입에는 사용하지 않습니다.

   ```java
    Set<?> s = Set.union(s1, s2);
   ```

2. 재귀적 타입 한정

   ```java
     public static <E extends Comparable<E>> E max(List<E> c)
   ```

   이 경우에는 매개변수 목록과 입력 매개변수 두 곳에 적용할 수 있습니다.

   * 매개변수 목록: `E extends Comparable<E>`

     원래 선언에서는 E가 Comparable\<E>를 확장한다고 정의했는데

     이 때, Comparable\<E>는 E 인스턴스를 소비합니다.

     그렇기 때문에 `? super E`를 사용합니다.

     <br>

   * 입력 매개변수: `List<E> c`

     입력 매개변수에서는 E 인스턴스를 생산하므로 `? extends E`를 사용합니다.

   ```java
     public static <E extends Comparable<? super E>> E max(List<? extends E> c)
   ```

## 5. 타입 매개변수 vs 와일드카드 타입

타입 매개변수와 와일드카드 타입은 공통 부분이 많아서 둘 중 어느 것을 사용해도 괜찮을 때가 있습니다.

그럼 둘 중 어떤 것을 선택해야 할까요?

그럴 경우에는 기본 규칙에 따라 설계할 수 있습니다.

> 🔅 기본 규칙 🔅
>
> 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드카드로 대체
>
> 1. 비한정적 타입 매개변수라면 비한정적 와일드카드 `?`
>
> 2. 한정적 타입 매개변수라면 한정적 와일드카드 `? extends E` `? super E`

## 6. 정리

이번 포스트는 타입 매개변수와 와일드카드를 비교해 보았습니다.

PECS 공식을 이용해 와일드카드를 적절하게 사용한다면 좀 더 유연한 API를 설계할 수 있으므로

PECS 공식을 잘 기억하도록 합시다.