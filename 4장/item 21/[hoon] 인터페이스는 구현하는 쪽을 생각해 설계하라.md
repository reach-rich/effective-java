## 1. 들어가기

Java 8 이전의 인터페이스는 메서드를 추가하면 기존 구현체에 컴파일 오류가 발생했기 때문에

기존 구현체를 깨뜨리지 않고는 메서드를 추가할 수 없었습니다.

그러나 Java 8 이후, 인터페이스는 새로운 봄을 맞이하게 되었습니다.

바로, '디폴트 메서드' 가 생기면서 기존 구현체를 깨뜨리지 않고 메서드를 추가할 수 있게 되었기 때문입니다.

이 디폴트 메서드를 사용하면 인터페이스의 모든 구현체에서 따로 재정의를 하지 않아도 사용할 수 있습니다.

하지만, 모든 상황에서 오류를 범하지 않는 디폴트 메서드를 작성하기란 쉽지 않습니다.

그럼 어떤 경우 오류 가능성이 있을까요?

## 2. 디폴트 메서드 추가로 오류 가능성이 있는 예시

디폴트 메서드는 컴파일에 성공하더라도 기존 구현체에 런타임 오류를 일으킬 수 있습니다.

대표적인 예로 아파치 라이브러리인 SynchronizedCollection가 있습니다.

SynchronizedCollection은 Collection 인터페이스를 구현했고,

SynchronizedCollection 인스턴스끼리 동기화를 적용한 클래스인데

Collection 인터페이스의 디폴트 메서드인 removeIf 메서드를 재정의하지 않고 있습니다.

```java
  default boolean removeIf(Predicate<? super E> filter) {
    Objects.requireNonNull(filter);
    boolean removed = false;
    final Iterator<E> each = iterator();
    while (each.hasNext()) {
      if (filter.test(each.next())) {
        each.remove();
        removed = true;
      }
    }
    return removed;
  }
```

그래서 모든 메서드 호출을 알아서 동기화해주지 못하는 문제점이 발생합니다.

따라서, SynchronizedCollection 인스턴스를 여러 스레드가 공유하는 환경에서 한 스레드가

removeIf 메서드를 호출하면 예기치 못한 결과로 이어질 가능성이 있습니다.

그렇기에 기존 인터페이스에 디폴트 메서드를 추가하는 일은 필요한 경우가 아니라면 지양해야 합니다.

## 3. 디폴트 메서드 추가로 오류 가능성 예방 조치 예시

Java 라이브러리의 Collections.SynchronizedCollection도 위의 아파치 SynchronizedCollection과 같이

SynchronizedCollection 인스턴스끼리 동기화를 적용한 클래스입니다.

하지만, Java 라이브러리는 위의 문제점을 예방하기 위해

동기화를 적용하는 방법으로 removeIf 메서드를 재정의했습니다.

```java
  @Override
  public boolean removeIf(Predicate<? super E> filter) {
    synchronized (mutex) {return c.removeIf(filter);}
  }
```

## 4. 정리

이번 포스트에서는 자바 8부터 생긴 디폴트 메서드에 대해 알아보았습니다.

디폴트 메서드는 기존 구현체를 깨뜨리지 않고 메서드를 추가할 수 있기 때문에

인터페이스를 더 편리하게 구현할 수 있는 장점이 있습니다.

하지만, 기존 인터페이스에 새로운 디폴트 메서드를 추가하는 경우 커다란 위험도 딸려오게 됩니다.

그래서 인터페이스 특히 디폴트 메서드를 설계 혹은 추가할 때에는 반드시 다양한 테스트를 거쳐야 합니다.