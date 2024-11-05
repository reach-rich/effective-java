Effective Java의 서른한 번째 아이템 "한정적 와일드카드를 사용해 API 유연성을 높이라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 1. <? extends T>

아이템 27의 Stack 클래스를 떠올려보자. 여기 Stack의 public API를 추려보았다.

```java
public class Stack<E> {
  public Stack();
  public void push(E e);
  public E pop();
  public boolean isEmpty();
}
```

여기에 일련의 원소를 스택에 넣는 메서드를 추가해야 한다고 해보자.

```java
public void pushAll(Iterable<E> src) {
  for (E e : src) {
    push(e);
  }
}
```

이 메서드는 깨끗이 컴파일되지만 완벽하지는 않다. Iterable src의 원소 타입이 스택의 원소 타입과 일치하면 잘 작동한다. 하지만 Stack\<Number\>로 선언한 후 Integer 타입의 intVal을 인자로 하여 pushAll(intVal)을 호출하면 어떻게 될까?

아래와 같은 오류 메시지가 뜬다. 매개변수화 타입이 불공변이기 때문이다.

```basic
StackTest.java:7: error: incompatible types: Iterable<Integer>
cannot be converted to Iterable<Number>
  numberStack.pushAll(integers);
                      ^
```

자바는 이런 상황에 대처할 수 있는 **한정적 와일드카드 타입**이라는 특별한 매개변수화 타입을 지원한다. pushAll의 입력 매개변수가 타입은 'E의 Iterable'이 아니라 'E의 하위 타입의 Iterable'이어야 하며, 와일드카드 타입 **Iterable\<? extends E\>**가 정확히 이런 뜻이다(이때 E의 하위 타입에는 E 자신도 포함된다).

```java
public void pushAll(Iterable<? extends E> src) {
  for (E e : src)
    push(e);
}
```

<br>

## 2. <? super T>

이제 pushAll과 짝을 이루는 popAll 메서드를 작성한다. popAll 메서드는 Stack 안의 모든 원소를 주어진 컬렉션으로 옮겨 담는다.

```java
public void popAll(Collection<E> dst) {
  while (!isEmpty())
    dst.add(pop());
}
```

이번에도 주어진 컬렉션의 원소 타입이 스택의 원소 타입과 일치한다면 말끔히 컴파일되고 문제없이 동작한다. 하지만 이번에도 역시 완벽하진 않다. 

Stack\<Number\>의 원소를 Object용 컬렉션으로 옮기려 한다고 해보자.

```java
Stack<Number> numberStack = new Stack<>();
Collection<Object> objects = ...;
numberStack.popAll(objects);
```

이 클라이언트 코드를 앞의 popAll 코드와 함께 컴파일하면 "Collection\<Obect\>는 Collection\<Number\>의 하위 타입이 아니다"라는 오류가 발생한다. 

이번에도 와일드카드 타입으로 해결할 수 있다. 이번에는 popAll의 입력 매개변수의 타입이 'E의 Collection'이 아니라 'E의 상위 타입의 Collection'이어야 한다(모든 상위 타입은 자기 자신의 상위 타입이다). 와일드카드 타입을 사용한 **Collection\<? super E\>**가 정확히 이런 의미다.

```java
public void popAll(Collection<? super E> dst) {
  while (!isEmpty())
    dst.add(pop());
}
```

<br>

## 3. PECS 공식

메시지는 분명하다. **유연성을 극대화하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하라.** **한편, 입력 매개변수가 생산자와 소비자 역할을 동시에 한다면 와일드카드 타입을 써도 좋을 게 없다.** 타입을 정확히 지정해야 하는 상황으로, 이때는 와일드카드 타입을 쓰지 말아야 한다.

다음 공식을 외워두면 어떤 와일드카드 타입을 써야 하는지 기억하는 데 도움이 될 것이다.

### 펙스(PECS) : producer-extends, consumer-super

PECS 공식은 와일드카드 타입을 사용하는 기본 원칙이다. 즉, 매개변수화 타입 T가 생산자라면 \<? extends T\>를 사용하고, 소비자라면 \<? super T\>를 사용하라. 

위의 Stack 예에서는 아래와 같이 공식이 적용되었다.

- pushAll의 src 매개변수 : Stack이 사용할 E 인스턴스를 생산하므로 src의 적절한 타입 Iterable\<? extends E\>이다.
- popAll의 dst 매개변수 : Stack으로부터 E 인스턴스를 소비하므로 dst의 적절한 타입 Collection\<? super E\>이다.



이 공식을 기억해주고, 앞선 아이템들에서 소개한 메서드와 생성자 선언을 다시 살펴보자. 

### 예제 1. Chooser

아이템 28의 Chooser 생성자는 다음과 같이 선언했다.

```java
public Chooser(Collection<T> choices)
```

이 생성자로 넘겨지는 choices 컬렉션은 T 타입의 값을 생산하기만 하니, T를 확장하는 와일드카드 타입을 사용해 선언해야 한다.

```java
public Chooser(Collection<? extends T> choices)
```

<br>

### 예제 2. union

아이템 30의 union 메서드는 다음과 같이 선언되었다.

```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2)
```

s1과 s2 모두 E의 생산자이니 PECS 공식에 따라 다음처럼 선언해야 한다.

```java
public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2)
```

제대로만 사용한다면 클래스 사용자는 와일드카드 타입이 쓰였다는 사실조차 의식하지 못할 것이다. 받아들여야 할 매개변수를 받고 거절해야 할 매개변수는 거절하는 작업이 알아서 이뤄진다. **클래스 사용자가 와일드카드 타입을 신경 써야 한다면 그 API에 무슨 문제가 있을 가능성이 크다.** 

> 반환 타입은 여전히 Set<E>임에 주목하자. 반환 타입에는 한정적 와일드카드 타입을 사용하면 안 된다. 유연성을 높여주기는커녕 클라이언트 코드에서도 와일드카드 타입을 써야 하기 때문이다.

<br>

### 예제 3. max

아이템 30의 max 메서드는 다음과 같이 선언되었다.

```java
public static <E extends Comparable<E>> E max(List<E> list)
```

다음은 와일드카드를 사용해 다듬은 모습이다.

```java
public static <E extends Comparable<? super E>> E max(List<? extends E> list)
```

이번에는 PECS 공식을 두 번 적용했다. 둘 중 더 쉬운 입력 매개변수 목록부터 살펴보자. 입력 매개변수에서는 E 인스턴스를 생산하므로 원래의 List\<E\>를 List\<? extends E\>로 수정했다.

다음은 더 난해한 쪽인 타입 매개변수 E다. 원래 선언에서는 E가 Comparable\<E\>를 확장한다고 정의했는데, 이때 Comparable\<E\>는 E 인스턴스를 소비한다. 그래서 매개변수화 타입 Comparable\<E\>를 한정적 와일드카드 타입인 Comparable\<? super E\>로 대체했다. **일반적으로 Comparable\<E\>보다는 Comparable\<? super E\>를 사용하는 편이 낫다. Comparator도 마찬가지다.**

다음 리스트는 오직 수정된 max로만 처리할 수 있다. 

```java
List<ScheduledFuture<?>> scheduledFutures = ..;
```

수정 전 max가 이 리스트를 처리할 수 없는 이유는 ScheduledFuture가 Comparable\<ScheduledFuture\>를 구현하지 않았기 때문이다. ScheduledFuture는 Delayed의 하위 인터페이스이고, Delayed는 Comparable\<Delayed\>를 확장했다. 다시 말해, ScheduledFuture의 인스턴스는 다른 ScheduledFuture 인스턴스뿐 아니라 Delayed 인스턴스와도 비교할 수 있어서 수정 전 max가 이 리스트를 거부하는 것이다.

따라서, Comparable을 직접 구현하지 않고, 직접 구현한 다른 타입을 확장한 타입을 지원하기 위해 와일드카드가 필요하다. 

<br>

## 4. 타입 매개변수 vs 와일드카드

타입 매개변수와 와일드카드에는 공통되는 부분이 있어서, 메서드를 정의할 때 둘 중 어느 것을 사용해도 괜찮을 때가 많다. 예를 들어 주어진 리스트에서 명시한 두 인덱스의 아이템들을 교환하는 정적 메서드를 두 방식 모두로 정의해보자. 첫 번째는 비한정적 타입 매개변수를 사용했고 두 번째는 비한정적 와일드카드를 사용했다.

```java
public static <E> void swap(List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j);
```

public API라면 간단한 두 번재 방법이 낫다. 어떤 리스트든 이 메서드에 넘기면 명시한 인덱스의 원소들을 교환해줄 것이다. 신경 써야 할 타입 매개변수도 없다.

기본 규칙은 이렇다. **메서드 선언에 타입 매개변수가 한 번만 나오면 와일드카드로 대체하라.** 이때 비한정적 타입 매개변수라면 비한정적 와일드카드로 바꾸고, 한정적 타입 매개변수라면 한정적 와일드카드로 바꾸면 된다. 

<br>

두 번째 swap 선언에는 문제가 하나 있는데, 다음과 같이 아주 직관적으로 구현한 코드가 컴파일되지 않는다는 것이다.

```java
public static void swap(List<?> list, int i, int j) {
  list.set(i, list.set(j, list.get(i)));
}
```

이 코드를 컴파일하면 그다지 도움이 되지 않는 오류 메시지가 나온다. 방금 꺼낸 원소를 리스트에 다시 넣을 수 없다니, 이게 대체 무슨 일인가? 원인은 리스트의 타입이 List\<?\>인데, List\<?\>에는 null 외에는 어떤 값도 넣을 수 없다는 데 있다. 

이 문제는 와일드카드 타입의 실제 타입을 알려주는 메서드들을 private 도우미 메서드로 따로 작성해 활용하여 해결할 수 있다. 실제 타입을 알아내려면 도우미 메서드는 제네릭 메서드여야 한다. 

```java
public static void swap(List<?> list, int i, int j) {
  swapHelper(list, i, j);
}

private static <E> void swapHelper(List<E> list, int i, int j) {
  list.set(i, list.set(j, list.get(i)));
}
```

swapHelper 메서드는 리스트가 list\<E\>임을 알고 있다. 즉, 이 리스트에서 꺼낸 값의 타입은 항상 E이고, E 타입의 값이라면 이 리스트에 넣어도 안전함을 알고 있다. 

<br>

## 5. 핵심 정리

- 조금 복잡하더라도 와일드카드 타입을 적용하면 API가 훨씬 유연해진다. 그러니 널리 쓰일 라이브러리를 작성한다면 반드시 와일드카드 타입을 적절히 사용해줘야 한다.
- PECS 공식을 기억하자. 즉, 생산자(producer)는 extends 소비자(consumer)는 super를 사용한다.

<br>

## 6. Related Posts

- 불공변 (Item 28)
- 리스코프 치환 원칙 (Item 10)
- 비한정적 타입 매개변수 (Item 30)
