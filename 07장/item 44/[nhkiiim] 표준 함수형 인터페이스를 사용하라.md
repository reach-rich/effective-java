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
그 중 기본 함수형 인터페이스 6개에 대해 알아보자.

- UnaryOperator : 반환값과 인수의 타입이 같은 함수 (인수값이 하나)
- BinaryOperator : 반환값과 인수의 타입이 같은 함수 (인수값이 두개)
- Predicate : 인수값 하나를 받아 boolean 반환
- Function : 인수값과 반환타입이 다른 함수
- Supplier : 인수를 받지 않고 값을 반환하는 함수
- Consumer : 인수값을 하나 받고 반환값은 없는 함수


기본 인터페이스는 기본 타입인 int, log, double용으로 각 3개씩 변형이 생긴다.

Function 인터페이스는 기본 타입을 반환하는 변형이 총 9개가 더 있다.

그리고 다양한 변형들이 존재 하지만 다 외우기엔 43개가 있고 규칙성도 부족하기 때문에 필요할 때 찾아서 잘 사용하도록 하자.


#
### 3. 주의 사항

__(1) 기본 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용하지 말자__

표준 함수형 인터페이스 대부분은 기본 타입만 지원한다. 

그렇다고 기본 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용한다면 계산량이 많을 때 성능 저하를 일으킬 수 있기 때문에 지양해야 한다.

<br>

__(2) 직접 전용 함수형 인터페이스를 생성해야하는 경우__

대부분의 상황에서는 표준 함수형 인터페이스를 사용하는 것이 좋지만, 표준 인터페이스 중 필요한 용도에 맞는게 없다면 직접 생성하라

매개변수를 3개 받는 Predicate와 같이 구조적으로 똑같은 표준 함수형 인터페이스가 있더라도 직접 작성해야하는 경우가 있을 수도 있다.

다름의 조건 중 하나 이상을 만족한다면 전용 함수형 인터페이스를 구현해야하는지 고민해볼 필요가 있다.

> - 자주 쓰이며 이름 자체가 용도를 명확히 설명해준다.
> - 반드시 따라야하는 규약이 있다.
> - 유용한 디폴트 메서드를 제공할 수 있다.

<br>

__(3) 직접 만든 함수형 인터페이스에는 항상 @FunctionalInterface 어노테이션을 사용하라__

- 해당 클래스의 코드나 설명 문서에서 인터페이스가 람다용으로 설계된 것임을 알려주기 위한 역할

- 해당 인터페이스가 추상 메서드를 하나만 가지고 있어야 컴파일 가능하게 해줌

- 유지보수 과정에서 메서드를 추가하지 못하게 막아줌


<br>


__(4) 서로 다른 한수형 인터페이스를 같은 위치의 인수로 받는 메서드는 다중정의하지 마라__

사용자가 사용하기 모호한 상황이 발생할 수 있으며 올바른 메서드를 알려주기 위해 형변환해야하는 상황이 발생할 수 있다.







