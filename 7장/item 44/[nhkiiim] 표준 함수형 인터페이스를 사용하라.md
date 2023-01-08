# 표준 함수형 인터페이스를 사용하라

자바가 `람다`를 지원하면서 API를 작성하는 모범 사례도 바뀌었다.

이전에는 상위 클래스의 기본 메서드를 재정의해 원하는 동작을 구현하는 템플릿 메서드 패턴을 많이 사용했다면,

이제는 함수 객체를 받는 정적 팩터리나 생성자를 제공하는 방법을 많이 사용한다.

함수 객체를 매개변수로 받는 생성자와 메서드를 더 많이 만들고 함수형 매개변수의 타입을 올바르게 선택해야 한다.

#
### 1. 표준 함수형 인터페이스를 사용하자

필요한 용도에 맞는 표준 함수가 있다면, 직접 구현하지 말고 `java.util.function` 패키지의 표준 함수형 인터페이스를 활용하자

__(1) 기본 메서드 재정의 예시__

LinkedHashMap의 removeEldestEntry 재정의하면 put 메서드가 호출해 원소 100개를 유지하는 캐시로 사용할 수 있다.

```java
protected boolean removeEldestEntry(Map.Entry<K,V> eldest){
  return size() > 100;
}
```

<br>

__(2) 함수형 인터페이스 만들어 사용하기__

인터페이스를 만들어 사용해도 잘 동작하지만, 자바 표준 라이브러리에 이미 같은 모양의 인터페이스가 존재하기 때문에 직접 구현할 필요가 없다.

```java
@FunctionalInterface interface EldestEntryRemovalFunction<K,V>{
  boolean remove(Map<K,V> map, Map.Entry<K,V> eldest);
}
```


<br>

__(3) 표준 함수형 인터페이스 사용__
```java
// BiPredicate<Map<K,V>, Map.Entry<K,V>>
@FunctionalInterface
public interface BiPredicate<T, U> {
  boolean test(T t, U u);

  default BiPredicate<T, U> and(BiPredicate<? super T, ? super U> other) {
    Objects.requireNonNull(other);
    return (T t, U u) -> test(t, u) && other.test(t, u);
  }

  default BiPredicate<T, U> negate() {
    return (T t, U u) -> !test(t, u);
  }

  default BiPredicate<T, U> or(BiPredicate<? super T, ? super U> other) {
    Objects.requireNonNull(other);
    return (T t, U u) -> test(t, u) || other.test(t, u);
  }
}
```

#
### 2. 표준 함수형 인터페이스
`java.util.function`에는 총 43개의 인터페이스가 존재한다.
그 중 기본인터페이스 6개에 대해 알아보자.

- UnaryOperator : 반환값과 인수의 타입이 같은 함수 (인수값이 하나)
- BinaryOperator : 반환값과 인수의 타입이 같은 함수 (인수값이 두개)
- Predicate : 인수값 하나를 받아 boolean 반환
- Function : 인수값과 반환타입이 다른 함수
- Supplier : 인수를 받지 않고 값을 반환하는 함수
- Consumer : 인수값을 하나 받고 반환값은 없는 함수


기본 인터페이스는 기본 타입인 int, log, double용으로 각 3개씩 변형이 생긴다.

Function 인터페이스는 기본 타입을 반환하는 변형이 총 9개가 더 있다.





