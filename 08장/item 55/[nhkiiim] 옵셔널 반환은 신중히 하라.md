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


#### 💡 스트림 종단 연산과 옵셔널

> 스트림의 종단 연산 중 상당수가 옵셔널을 반환한다!

```java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
  return c.stream().max(Comparator.naturalOrder());
}
```
- 스트림의 max 연산이 옵셔널을 생성


#
### 옵셔널의 반환값

> 옵셔널은 검사 예외와 취지가 비슷함 -> 반환값이 없을 수도 있음을 API 사용자에게 명확하게 알려줌

#### 1) 기본값 정해두기
```java
String lastWordInLexicon = max(words).orElse("단어 없음...");
```

#### 2) 상황에 맞는 예외 던지기
```java
// 실제 예외가 아닌 예외 팩터리 건네기 (예외가 발생하지 않으면 예외 생성 비용 들지 않음)
Tot myToy = max(toys).orElseThrow(TemperTantrumException::new); 
```

#### 3) (항상 값이 채워져 있다고 확신한다면) 값 바로 꺼내기
```java
Element lastNobaleGas = max(Elements.NOBLE_GASES).get();
```


> 기본값을 설정하는 비용이 커 부담 될 경우에는 `Supplier<T>`를 인수로 받는 `orElseGet`을 사용하면 초기 설정 비용을 낮출 수 있다!

#### +) filter, map, flatMap, ifPresent 등 다양한 메서드 활용도 가능
> https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html

<br>

#### 💡 isPresent 메서드
- 옵셔널이 채워져 있으면 true, 비어있으면 false 반환
- 원하는 모든 작업을 수행할 수 있지만 신중히 사용해야함 (앞서 언급한 메서드들로 대체 가능)

#
### 옵셔널을 반환해야하는 경우
- __결과가 없을 수 있으며, 클라이언트가 이 상황을 특별하게 처리해야할 경우 `Optional<T>` 반환__
- 옵셔널도 새로 할당해 초기화해야하는 객체이고, 또 값을 꺼내는 연산이 필요하기 때문에 성능에 영향을 미칠 수도 있음
- 성능이 중요한 상황을 판단하기 위해서는 세심히 측정해보는 수 밖에 없음..!


#
### 옵셔널 사용 주의사항
> 반환값으로 옵셔널을 사용하는 것이 무조건 좋은 건 아니다!

- __컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸면 안됨__
- `Optional<List<T>>` 보다는 빈 `List<T>`를 반환하는 것이 바람직 (옵셔널 처리 코드 필요 없음)

<br>

- int, long, double은 박싱된 기본타입으로 옵셔널을 사용하면 두겹이 감싸져 무거워 지기 때문에 전용 옵셔널이 존재 (OptionalInt, OptionalDouble, OptionalLong)
- 박싱된 기본 타입을 담은 옵셔널은 사용하지 말기! (덜 중요한 기본타입인 Boolean, Byte, Character, Short, Float는 예외일 수 있음)

<br>

- 여기서 언급하지 않은 쓰임으로 옵셔널을 사용하는 것은 적절하지 않음
- __옵셔널을 컬렉션의 키, 값, 원소나 배열의 원소로 사용하는 경우는 거의 없음__ (맵에도 사용 금지)

<br>

- 옵셔널을 인스턴스 필드에 저장하는 상황이 있을 수도 있으나, 복잡할 것! (필수 필드를 가지는 상위 클래스를 확장하는데, 하위 필드에서는 선택적 필드로 사용하는 나쁜 냄새가 남!)
