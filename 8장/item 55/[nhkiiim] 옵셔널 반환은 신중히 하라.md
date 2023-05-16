## 옵셔널 반환은 신중히 하라

### 반환값이 없을 때!

#### 1) 자바 8 이전

자바8 이전에 메서드가 특정 조건에서 값을 반환할 수 없을 때 취할 수 있는 선택지

> 1. 예외 던지기

- 예외는 진짜 예외적인 상황에서만 사용해야함
- 예외 생성 시 스택 추적 전체를 캡쳐하기 때문에 비용도 만만치 않음

> 2. (반환 타입이 객체라면) null 반환하기

- null 반환 가능성이 있는 메서드 호출 시, 별도의 null 처리 코드 투가 필요
- null 처리를 무시하고 반환된 null을 어딘가에 저장해두면 전혀 관련 없는 메서드에서 `NullPointerException` 발생 가능


#### 2) 자바 8 이후
> `Optional<T>` 라는 선택지가 하나 더 생김

#
### Optional
null이 아닌 T 타입을 참조하거나, 아무것도 담지 않을 수 있음

- 옵셔널은 원소를 최대 1개 가질 수 있는 __불변 컬렉션__ (`Optional<T>`가 `Collection<T>`를 구현하지는 않았지만 원칙적으로는 그렇다!)
- 아무것도 반환하지 않아야 할 때 T 대신 `Optional<T>` 반환 가능
- 예외를 던지는 메서드보다 유연하고 사용하기 쉽고, null을 반환하는 메서드보다 오류 가능성이 작음!! (짱)

> 옵셔널을 반환하는 예시
```java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
  if (c.isEmpty()) {
    return Optional.empty();
  }
  
  E result = null;
  
  for (E e: c) {
    if (result == null || e.compareTo(result) > 0) {
      result = Objects.requireNonNull(e);
    }
  }
  
  return Optional.of(result);
}
```

- `Optional.empty()` : 빈 옵셔널 만들기
- `Optional.of(result)` : 값이 든 옵셔널 만들기 (null을 넣으면 NPE 발생)

__옵셔널을 반환하는 메서드에서는 절대 null을 반환하지 말자 (옵셔널 도입 취지를 완전 무시하는 행위)__


