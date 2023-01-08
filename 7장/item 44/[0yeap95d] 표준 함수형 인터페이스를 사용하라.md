### 📝 표준 함수형 인터페이스

- 자바가 람다 지원하면서 템플릿 메서드 패턴 줄어들었음
- 같은 효과의 함수 객체를 받는 정적 팩터리나 생성자를 제공하는 방법으로 대체
  - 함수 객체를 매개변수로 받는 생성자와 메서드를 더 많이 만들어야함
  - 함수형 매개변수 타입을 올바르게 선택해야 함

<br>

**✏ #01 예제소스 | LinkedHashMap클래스에서 removeEldestEntry를 재정의하면 캐시로 사용 가능**

```java
protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
    return size() > 100;
}
```

>원소가 100개가 될때까지 커지다가, 그 이상이 되면 키가 더해질때마다 가장 오래된 원소 하나씩 제거
>
>`Map.Entry<K,V>`를 받아 `boolean`을 반환해야할 것 같지만, 꼭 그렇지는 않음
>
>`size()`를 호출해 맵 안의 원소 수를 알아내는데, 해당 메서드가 인스턴스 메서드라 가능한 방식

- 생성자에 넘기는 함수 객체는 이 맵의 인스턴스 메서드가 아님
- 팩터리나 생성자를 호출할 때는 맵의 인스턴스가 존재하지 않기 때문에 맵은 자기 자신도 함수 객체에 건네줘야 함

<br>

**✏ #02 예제소스 | 불필요한 함수형 인터페이스 - 대신 표준 함수형 인터페이스를 사용하라**

```java
@FunctionalInterface interface EldestEntryRemovalFunction<K, V> {
    boolean remove(Map<K, V> map, Map.Entry<K,V> eldest);
}
```

>자바 표준 라이브러리에 이미 같은 모양의 인터페이스가 준비되어 있음
>
>java.util.function 패키지를 보면 다양한 용도의 표준 함수형 인터페이스가 존재

<br>

**✔ 필요한 용도에 맞는 게 있다면, 직접 구현하지 말고 표준 함수형 인터페이스를 활용하라**

- API 다루는 개념의 수가 줄어들어 익히기 쉬움
- 표준 함수형 인터페이스들은 유용한 디폴트 메서드를 많이 제공하므로 다른 코드와의 상호운용성도 좋아짐

<br>

---

### 📚 java.util.function 인터페이스

- 패키지에 총 43개의 인터페이스가 존재
- 기본 인터페이스 6개만 기억하면 나머지 충분히 유추 가능
- 기본 인터페이스들은 모두 참조 타입용

<br>

**Operator인터페이스** : 인수가 1개인 UnaryOperator와 2개인 BinaryOperator로 나뉘며, 반환값과 인수의 타입이 같은 함수

**Predicate인터페이스 **: 인수 하나를 받아 boolean을 반환하는 함수를 뜻하며, Function 인터페이스는 인수와 반환 타입이 다른 함수

**Supplier인터페이스** : 인수를 받지 않고 값을 반환(혹은 제공)하는 함수

**Consumer인터페이스** : 인수를 하나 받고 반환값은 없는(특히 인수를 소비하는) 함수

<br>

**🔍 기본 함수형 인터페이스**

| 인터페이스          | 함수 시그니처         | 예                    |
| ------------------- | --------------------- | --------------------- |
| `UnaryOpertor<T>`   | `T apply(T t)`        | `String::toLowerCase` |
| `BinaryOperator<T>` | `T apply(T t1, T t2)` | `BigInter::add`       |
| `Predicate<T>`      | `boolean test(T t)`   | `Collection::isEmpty` |
| `Function<T>`       | `R apply(T t)`        | `Arrays::asList`      |
| `Supplier<T>`       | `T ger()`             | `Instant::now`        |
| `Consumer<T>`       | `void accept(T t)`    | `System.out::println` |

<br>

**➕ 기본 인터페이스에서 기본 타입 int, long, double용으로 각 3개씩 변형**

- 예시 `IntPredicate`, `LongBinaryOperator`, ...
- `Function`의 변형만 매개변수화 되어있음
  - `LongFunction<int[]>` 은 `long` 인수를 받아 `int[]`를 반환

<br>

**➕ Function 인터페이스에는 기본 타입을 반환하면 변형이 총 9개 더 있음**

- `Function` 인터페이스의 변형은 입력과 결과의 타입이 항상 다름
- 입력과 결과 타입이 모두 기본 타입이면 접두어로 `SrcToResult`를 사용 (6개)
  - `LongToIntFunction` : `long`을 받아 `int`를 반환
- 입력이 객체 참조이고 결과가 `int`, `long`, `double`인 경우 `ToResult`를 사용 (3개)
  - `ToLongFunction<int[]>` : `int[]`를 인수로 받아 `long`을 반환

<br>

**➕ 기본 함수형 인터페이스 중 인수를 2개씩 받는 변형이 3개 있음**

- `BiPredicate<T,U>`, `BiFunction<T,U,R>`, `BiConsumer<T,U>` 변형이 존재

  - `ToIntBiFunction<T,U>`, `ToLongBiFunction<T,U>`, `ToDoubleBiFunction<T,U>` 변형이 존재

  + `ObjDoubleBiConsumer<T>` , `ObjIntBiConsumer<T>`,  `ObjLongBiConsumer<T>` 변형이 존재

<br>

**➕ BooleanSupplier 인터페이스**

- `boolean`을 반환하도록한 `Supplier`의 변형

---

<br>

### 📌 핵심정리

**이제 자바도 람다를 지원한다**

**여러분도 지금부터는 API를 설계할 때 람다도 염두에 두어야 한다는 뜻이다**

**입력값과 반환값에 함수형 인터페이스 타입을 활용하라**

**보통은 java.util.function 패키지의 표준 함수형 인터페이스를 사용하는 것이 가장 좋은 선택이다**

**단, 흔치는 않지만 직접 새로운 함수형 인터페이스를 만들어 쓰는 편이 나을 수도 있음을 잊지 말자**
