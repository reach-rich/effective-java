# 표준 함수형 인터페이스를 사용하라

자바가 `람다`를 지원하면서 API를 작성하는 모범 사례도 바뀌었다.

이전에는 상위 클래스의 기본 메서드를 재정의해 원하는 동작을 구현하는 템플릿 메서드 패턴을 많이 사용했다면,

이제는 함수 객체를 받는 정적 팩터리나 생성자를 제공하는 방법을 많이 사용한다.

함수 객체를 매개변수로 받는 생성자와 메서드를 더 많이 만들고 함수형 매개변수의 타입을 올바르게 선택해야 한다.

#
### 1. 표준 함수형 인터페이스를 사용하자

필요한 용도에 맞는 표준 함수가 있다면, 직접 구현하지 말고 `java.util.function` 패키지의 표준 함수형 인터페이스를 활용하자

__(1) 이전 예시__
```java
//LinkedHashMap의 removeEldestEntry : put 메서드가 호출해 원소 100개를 유지한다
protected boolean removeEldestEntry(Map.Entry<K,V> eldest){
  return size() > 100;
}
```

__(2) 불필요한 함수형 인터페이스 예시__
```java
@FunctionalInterface interface EldestEntryRemovalFunction<K,V>{
  boolean remove(Map<K,V> map, Map.Entry<K,V> eldest);
}
```

(2)의 인터페이스도 잘 동작하지만, 자바 표준 라이브러리에 이미 같은 모양의 인터페이스가 존재하기 때문에 직접 구현할 필요가 없다.

__(3) 표준 함수형 인터페이스 예시__
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

