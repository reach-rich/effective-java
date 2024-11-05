## 1. 들어가기

Item 18에서는 상속을 염두에 두지 않고 설계한 클래스를 상속했을 때의 문제점을 살펴보았습니다.

그럼 상속을 염두에 둔 클래스를 어떻게 판단할 수 있을까요? 🤔

이번 Item에서는 어떤 클래스를 상속해야 하는지 알아보겠습니다.

## 2. 상속을 염두에 둔 클래스를 판단하는 방법

> 🎯 메서드의 사용법을 문서로 남긴 클래스

상속을 염두에 둔 클래스는 대표적으로 재정의 가능한 메서드들의 사용법을 문서로 남겨놓습니다.

대표적인 예로 java.util.AbstractCollection의 remove 메서드가 있습니다.

```java
  /**
    * {@inheritDoc}
    *
    * @implSpec
    * This implementation iterates over the collection looking for the
    * specified element.  If it finds the element, it removes the element
    * from the collection using the iterator's remove method.
    *
    * <p>Note that this implementation throws an
    * {@code UnsupportedOperationException} if the iterator returned by this
    * collection's iterator method does not implement the {@code remove}
    * method and this collection contains the specified object.
    *
    * @throws UnsupportedOperationException {@inheritDoc}
    * @throws ClassCastException            {@inheritDoc}
    * @throws NullPointerException          {@inheritDoc}
    */
  public boolean remove(Object o) { ... }
```

API 문서의 메서드 설명 끝에 종종 "Implementation Requirements"로 시작하는 절을 볼 수 있는데

그 부분이 메서드의 내부 동작 방식을 설명하는 부분입니다.

remove 메서드에서는 아래 부분이 Implementation Requirements에 해당됩니다.

```
  @implSpec
  This implementation iterates over the collection looking for the
  specified element.  If it finds the element, it removes the element
  from the collection using the iterator's remove method.

  <p>Note that this implementation throws an
  {@code UnsupportedOperationException} if the iterator returned by this
  collection's iterator method does not implement the {@code remove}
  method and this collection contains the specified object.
```

iterator 메서드가 반환한 반복자가 remove 메서드를 구현하지 않는다면 예외를 발생할 수 있다는 뜻으로

iterator 메서드를 재정의하면 remove 메서드의 동작에 영향을 미치는 것을 알 수 있습니다.

> 🎯 클래스 내부 동작 과정에 끼어들 수 있는 훅(hook)을 선별해 protected 메서드로 공개한 클래스

대표적인 예로 java.util.AbstractList의 removeRange 메서드가 있습니다.

```java
  /**
    * Removes from this list all of the elements whose index is between
    * {@code fromIndex}, inclusive, and {@code toIndex}, exclusive.
    * Shifts any succeeding elements to the left (reduces their index).
    * This call shortens the list by {@code (toIndex - fromIndex)} elements.
    * (If {@code toIndex==fromIndex}, this operation has no effect.)
    *
    * <p>This method is called by the {@code clear} operation on this list
    * and its subLists.  Overriding this method to take advantage of
    * the internals of the list implementation can <i>substantially</i>
    * improve the performance of the {@code clear} operation on this list
    * and its subLists.
    *
    * @implSpec
    * This implementation gets a list iterator positioned before
    * {@code fromIndex}, and repeatedly calls {@code ListIterator.next}
    * followed by {@code ListIterator.remove} until the entire range has
    * been removed.  <b>Note: if {@code ListIterator.remove} requires linear
    * time, this implementation requires quadratic time.</b>
    *
    * @param fromIndex index of first element to be removed
    * @param toIndex index after last element to be removed
    */
  protected void removeRange(int fromIndex, int toIndex) { ... }
```

해당 코드에서 주의 깊게 봐야할 부분은 다음과 같습니다.

```
  <b>Note: if {@code ListIterator.remove} requires linear
  time, this implementation requires quadratic time.</b>
```

ListIterator.remove가 선형 시간이 걸리면 이 구현의 성능은 제곱에 비례한다는 뜻입니다.

List 구현체의 최종 사용자는 removeRange 메서드에 관심이 없습니다.

그럼에도 해당 메서드를 제공한 이유는 하위 클래스의 clear 메서드를 고성능으로 만들기 위해서입니다.

만약, removeRange 메서드가 없다면 하위 클래스에서 clear 메서드를 호출하면 성능이 떨어질 것입니다.

> 🎯 생성자 내부에서 재정의 가능 메서드를 호출하지 않는 클래스

바로 예시를 통해 알아보겠습니다.

```java
  public class Super {
    public Super() { overrideMethod(); }
    public void overrideMethod() {}
  }
```

```java
  public class Sub extends Super {
    private final Instant instant;

    Sub() { instant = Instant.now(); }

    @Override
    public void overrideMethod() {
      System.out.println(instant);
    }

    public static void main(String[] args) {
      Sub sub = new Sub();
      sub.overrideMethod();
    }
  }
```

현재 상위 클래스의 생성자는 재정의용 메서드를 호출하고 있습니다.

이 프로그램의 결과로 instant가 두 번 출력되리라 기대했으나,

하위 클래스가 생성되기 전 상위 클래스가 먼저 생성되기 때문에 첫 번째 출력은 null이 나옵니다.

만약 overrideMethod에서 instant를 참조한다면 NullPointerException이 발생할 수 있습니다.

> 🎯 Clonable이나 Serializable 인터페이스를 상속 받지 않는 클래스

Clonable이나 Serializable을 하나라도 구현한 클래스를 상속할 수 있게 설계하는 것은

그 클래스를 확장하려는 프로그래머에게 엄청난 부담을 지우기 때문에 좋지 않은 생각입니다.

대신, 이를 구현할 수 있도록 만드는 방법이 있습니다.

생성자 대신 clone이나 readObject 메서드를 사용하는 방법인데요.

이 두 메서드는 새로운 객체를 만든다는 점에서 생성자와 비슷한 효과를 냅니다.

그렇기 때문에 clone과 readObject 메서드 또한 재정의 가능 메서드를 호출해서는 안됩니다.

그리고 Serializable을 구현한 상속용 클래스가 readResolve나 writeReplace 메서드를 갖는다면

하위 클래스에서 무시되지 않게 하기 위해 private 대신 protected로 선언해야 합니다.

## 3. 상속용 클래스를 테스트하는 방법

앞서 상속을 염두에 둔 클래스를 판단하는 방법을 알아보았습니다.

그럼 우리가 상속용 클래스를 만들었을 때 그 클래스를 어떻게 테스트하면 좋을까요?

상속용 클래스로 적합한지 알려주는 도구를 사용하면 좋겠지만, 그런 도구는 아쉽게도 존재하지 않습니다.

가장 좋은 방법이자 유일한 방법은 "직접 하위 클래스를 만들어보는 것" 입니다.

꼭 필요한 protected 멤버를 놓쳤다면 하위 클래스를 작성할 때 빈자리가 확연히 드러납니다.

반대로, 여러 하위 클래스를 만들 때까지 전혀 사용하지 않는 protected 멤버는 private일 가능성이 큽니다.

## 4. 정리

클래스를 상속용으로 설계하려면 많은 노력이 들고 제약도 상당함을 알게 되었습니다.

그렇기 때문에 가장 좋은 방법은 상속용으로 설계하지 않은 클래스는 상속을 금지하는 것입니다.

상속을 금지하는 방법으로는 Item 17에서 알아보았듯이 두 가지가 있습니다.

* 클래스를 final로 선언한다.

* 모든 생성자를 private이나 package-private으로 선언하고 public 정적 팩터리를 만든다.

상속 대신 사용할 수 있는 방법은 Item 18에서 알아본 방법도 있습니다.

* Wrapper 클래스 패턴

만약, 꼭 상속을 허용하는 클래스를 만들어야 한다면

클래스 내부에서는 재정의 가능 메서드를 사용하지 않게 만들고

자기 사용 코드를 제거하라는 문서를 남겨야 합니다.