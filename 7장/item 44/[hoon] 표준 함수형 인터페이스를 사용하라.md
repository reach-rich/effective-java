## 1. 들어가기

람다가 도입되기 전에는 상위 클래스의 기본 메서드를 재정의해 원하는 동작을 구현하는

**템플릿 메서드 패턴**을 많이 사용했으나 람다 도입 이후부터는 매력이 크게 줄었습니다.

그 이유는 람다를 통해 함수 객체를 매개변수로 받는 것이 가능해졌기 때문입니다.

그럼, 기존의 템플릿 메서드 패턴을 어떻게 람다로 변경할 수 있을지 알아봅시다.

## 2. 템플릿 메서드 패턴을 람다로 대체하는 방법

먼저, 예시를 보겠습니다.

```java
  protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
    return size() > 100;
  }
```

해당 코드는 LinkedHashMap의 removeEldestEntry 메서드입니다.

Map에 새로운 키가 추가될 때 이 메서드를 호출해 `true`가 반환되면 가장 오래된 원소를 제거하는데요.

잘 동작하지만, 람다를 사용할 수도 있습니다.

그전에 removeEldestEntry 선언을 보면

`Map.Entry<K, V>`를 받아 `boolean`을 반환해야 할 것 같지만 꼭 그렇지는 않습니다.

왜냐하면 removeEldestEntry 내부의 `size()`는 인스턴스 메서드인데

생성자에 넘기는 함수 객체는 인스턴스 메서드가 아니기 때문에 자기 자신도 함수 객체에 넘겨야 합니다.

이를 반영하면 다음과 같은 함수형 인터페이스를 선언하고 사용할 수 있습니다.

```java
/* 함수형 인터페이스 선언 */
  @FunctionalInterface interface EldestEntryRemovalFunction<K, V> {
    boolean remove(Map<K, V> map, Map.Entry<K, V> eldest);
  }
```

```java
  /* 함수형 인터페이스 사용법 */
  public class Main {
    public static void main(String[] args) {
      Map<String, Integer> map = CacheMap.of((map1, eldest) -> map1.size() > 2);

      map.put("1", 1);
      map.put("2", 2);
      map.put("3", 3);

      System.out.println(map); // 2 , 3 출력
    }
  }

  class CacheMap<K, V> extends LinkedHashMap<K, V> {

    private EldestEntryRemovalFunction<K, V> function;

    /* 생성자 */
    private CacheMap(EldestEntryRemovalFunction<K, V> function) {
      this.function = function;
    }

    /* 정적 팩터리 메서드 */
    public static <K, V> CacheMap<K, V> of(EldestEntryRemovalFunction<K, V> function) {
      return new CacheMap<K, V>(function);
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
      return function.remove(this, eldest);
    }
  }
```

하지만, 이 인터페이스도 잘 동작하기는 하지만, 굳이 사용할 이유는 없습니다.

그 이유는 자바 표준 라이브러리에 이미 같은 모양의 표준 함수형 인터페이스가 있기 때문입니다.

```java
/* 표준 함수형 인터페이스(BiPredicate) 사용법 */
  public class Main {
    public static void main(String[] args) {
      Map<String, Integer> map = CacheMap.of((map1, eldest) -> map1.size() > 2);

      map.put("1", 1);
      map.put("2", 2);
      map.put("3", 3);

      System.out.println(map); // 2 , 3 출력
    }
}

class CacheMap<K, V> extends LinkedHashMap<K, V> {

    private BiPredicate<Map<K, V>,Map.Entry<K, V>> function;

    /* 생성자 */
    private CacheMap(BiPredicate<Map<K, V>,Map.Entry<K, V>> function) {
      this.function = function;
    }

    /* 정적 팩터리 메서드 */
    public static <K, V> CacheMap<K, V> of(BiPredicate<Map<K, V>,Map.Entry<K, V>> function) {
      return new CacheMap<>(function);
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
      return function.test(this, eldest);
    }
}
```

## 3. 표준 함수형 인터페이스

표준 함수형 인터페이스는 총 43개의 인터페이스가 있습니다.

하지만, 전부 기억하기엔 어렵기 때문에 기본 인터페이스 6개에서 유추하는 방법으로 사용할 수 있습니다.

기본 인터페이스는 다음과 같습니다.

* `UnaryOperator<T>`

  - 반환 값과 인수 타입이 같은 함수 (인수가 1개)

  - `T apply(T t)`

* `BinaryOperator<T>`

  - 반환 값과 인수 타입이 같은 함수 (인수가 2개)

  - `T apply(T t1, T t2)`

* `Predicate<T>`

  - 인수 하나를 받아 boolean을 반환하는 함수

  - `boolean test(T t)`

* `Function<T, R>`

  - 인수와 반환 타입이 다른 함수

  - `R apply(T t)`

* `Supplier<T>`

  - 인수를 받지 않고 값을 반환하는 함수

  - `T get()`

* `Consumer<T>`

  - 인수 하나를 받고 반환 값은 없는 함수

  - `void accept(T t)`

## 4. 커스텀 함수형 인터페이스를 구현해야 하는 경우

지금까지만 놓고 봤을 때 대부분의 경우 표준 함수형 인터페이스를 사용하는 것이 더 나아보입니다.

그럼 언제 함수형 인터페이스를 직접 구현해야 할까요?

1. 표준 인터페이스 중 필요한 용도에 맞는 구조가 없는 경우

   예를 들어 매개변수 3개를 받는 `Predicate`는 표준 함수형 인터페이스에 없기 때문에

   직접 구현해야 합니다.

   <br>

2. 예외 사항

   구조적으로 동일하지만, 직접 구현해야 하는 경우가 있습니다.

   * 자주 쓰이며, 이름 자체가 용도를 명확히 설명해준다.

   * 반드시 따라야 하는 규약이 있다.

   * 유용한 디폴트 메서드를 제공할 수 있다.

## 5. 커스텀 함수형 인터페이스 주의사항

1. @FunctionalInterface 애너테이션을 사용하자.

   해당 애너테이션을 사용하는 이유는 @Override를 사용하는 이유와 비슷합니다.

   * 해당 인터페이스를 사용할 사람에게 람다용으로 설계된 것임을 알려준다.

   * 추상 메서드를 오직 하나만 가지고 있어야 컴파일되게 해준다.

   * 유지보수 과정에서 누군가 실수로 메서드를 추가하지 못하게 막아준다.

## 6. 함수형 인터페이스 주의사항

1. 서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서드들을 다중 정의하지 말자.

   클라이언트에게 불필요한 모호함만 안겨주고, 이로 인해 실제로 문제가 일어납니다.

   ex. ExecutorService의 submit 메서드는 `Callable<T>`와 `Runnable` 다중 정의

   → 올바른 메서드를 알려주기 위해 형변환 필요

   <br>

2. 기본 타입 함수형 인터페이스에 박싱 타입을 사용하지 말자.

   박싱 타입을 사용한다면 특히 계산량이 많을 때는 성능이 처참히 느려질 수 있습니다.

## 7. 정리

이번 포스트는 함수형 인터페이스에 대해 알아보았습니다.

Java 8부터는 람다를 지원하기 때문에 지금부터는 API 설계 시 람다도 염두해야 합니다.

대부분의 경우는 표준 함수형 인터페이스를 사용하면 되나,

특정 경우에는 커스텀 함수형 인터페이스를 사용해야 하므로 상황에 맞게 함수형 인터페이스를 정의합시다.