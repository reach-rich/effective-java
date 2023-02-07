### 💡 스트림 패러다임

**: 스트림은 그저 또 하나의 API가 아닌, 함수형 프로그래밍에 기초한 패러다임**

<br>

**스트림 패러다임의 핵심**

- 계산을 일련의 변환으로 재구성하는 부분

- 각 변환 단계는 가능한 한 이전 단계의 결과를 받아 처리하는 **순수 함수**여야 함

  🔎***순수 함수**란 오직 입력만이 결과에 영향을 주는 함수로 다른 가변상태를 참조하지 않고, 스스로도 다른 상태를 변경하지 않음*

- 스트림 연산에 건네는 함수 객체는 모든 부작용(side effect)이 없어야 함

<br>

**✏ #01 예제소스 | 스트림 패러다임을 이해하지 못한 채 API만 사용**

```java
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
    words.forEach(word -> {
        freq.merge(word.toLowerCase(), 1L, Long::sum);
    });
}
```

>스트림, 람다, 메서드 참조를 사용했고, 결과도 올바르지만 스트림 코드라 할 수 없음
>
>스트림을 가장한 반복적 코드
>
>스트림 API의 이점을 살리지 못해 기능의 반복적 코드보다 길고, 일기 어렵고, 유지보수도 좋지 않음
>
>이 코드의 모든 종단 연산인 forEach에서 일어나는데, 이때 외부 상태(빈도표)를 수정하는 람다를 실행하면서 문제 발생
>
>forEach가 그저 스트림이 수행한 연산 결과를 보여주는 일 이상을 하는 것 (람다가 상태를 수정)

<br>

**✏ #02 예제소스 | 스트림을 제대로 활용해 빈도표를 초기화**

```java
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
    freq = words
        .collect(groupingBy(String::toLowerCase, counting()));
}
```

>스트림 API를 제대로 사용하고 있으며, 짧고 명확함

<br>

**✔forEach 연산은 스트림 계산 결과를 보고할 때만 사용하고, 계산하는 데는 쓰지 말 것**

<br>

---

<br>

### 📚 java.util.stream.Collectors 클래스

<br>

**1️⃣ collector(수집기)**

- 수집기가 생성하는 객체는 일반적으로 컬렉션이며, 따라서 `collector`라는 이름을 사용
- 수집기는 `toList()`, `toSet()`, `toCollection(collectionFactory)` 세 가지
- 리스트, 집합, 프로그래머가 지정한 컬렉션 타입을 반환

<br>

**✏ #03 예제소스 | 빈도표에서 가장 흔한 단어 10개를 뽑아내는 파이프라인**

```java
List<String> topTen = freq.keySet().stream()
    .sorted(comparing(freq::get).reversed())
    .limit(10)
    .collect(toList());
```

>**마지막 `toList`는 `Collectors`의 메서드로 `Collectors`의 멤버를 정적 임포트하여 쓰면 스트림 파이프 라인 가독성이 좋아져서 흔이 사용하는 방법**
>
>`comparing` 메서드는 키 추출 함수를 받는 비교자 생성 메서드
>
>`freq::get` 입력받은 단어(키)를 빈도표에서 찾아(추출) 그 빈도를 반환
>
>`reversed()` 가장 흔한 단어가 위로 오도록 역순으로 정렬

<br>

**2️⃣ toMap**

- `toMap(keyMapper, valueMapper)` 형태로 사용

**✏ #04 예제소스 | toMap 수집기를 사용하는 문자열을 열거 타입 상수에 매핑 **

```java
private static final Map<String, Operation> stringToEnum =
    Stream.of(values()).collect(
		toMap(Object::toString, e -> e));
```

>`toMap` 형태는 스트림의 각 원소가 고유한 키에 매핑되어 있을 때 적합
>
>스트림 원소 다수가 같은 키를 사용하는 경우 파이프라인이 `IllegalStateException` 던지며 종료
>
>아래에 이러한 충돌을 다루는 예시로 병합 함수의 형태 `BinaryOperator<U>`를 제공

<br>

**✏ #05 예제소스 | 각 키와 해당 키의 특정 원소를 연관 짓는 맵을 생성하는 수집기**

```java
Map<Artist, Album> topHits = albums.collect(
	toMap(Album::artist, a->a, maxBy(comparing(Album::sales))));
```

>`BinaryOperator`에서 정적 임포트한 `maxBy`라는 정적 팩터리 메서드 사용
>
>`maxBy`는 `Comparator<T>`를 입력받아 `BinaryOperator<T>`를 반환
>
>`comapring`이 `maxBy`에 넘겨줄 비교자를 반환하는데, 자신의 키 추출 함수로 `Album::sales`를 받음
>
>**즉, 앨범 스트림을 맵으로 바꾸는데, 이 맵은 각 음악가와 그 음악가의 베스트 앨범을 짝지은 것**

<br>

**✏ #06 예제소스 | 마지막에 쓴 값을 취하는 수집기**

```java
toMap(keyMapper, valueMapper, (oldVal, newVal) -> newVal)
```

>`toMap`을 이용한 충돌이 나면 마지막 값을 취하는 수집기 생성

<br>

**3️⃣ groupingBy**

- 입력으로 분류 함수를 받고 출력으로는 원소들을 카테고리별로 모아 놓은 맵을 담은 수집기를 반환
- 분류 함수는 입력받은 원소가 속하는 카테고리를 반환
- 이 카테고리가 해당 원소의 맵 키로 사용

```java
words.collect(groupingBy(word -> alphabetize(word)))
```

<br>

- 리스트 외의 값을 갖는 맵을 생성하려면, 분류 함수와 함께 **다운스트림(downstream)** 수집기도 명시

  >다운스트림 수집기는 해당 카테고리의 모든 원소를 담은 스트림으로부터 값을 생성
  >
  >1. `toSet()`을 넘기는 것으로 리스트가 아닌 집합을 값으로 가지는 맵을 생성
  >2. `toCollection(collectionFactory)` 넘기는 것으로 리스트나 집합 대신 컬렉션을 값으로 가지는 맵을 생성
  >3. `counting()`을 건네는 방법으로 각 카테고리를 해당 카테고리에 속하는 원소의 개수와 맵핑한 맵을 생성
  >
  >```java
  >Map<String, Long> freq = words
  >    .collect(groupingBy(String::toLowerCase, counting()));
  >```

<br>

- **다운스트림**와 **맵 팩터리** 지정

  >`mapFactory` 매개변수가 `downStream` 매개변수보다 앞에 놓임
  >
  >맵과 그 안에 담긴 컬렉션의 타입을 모두 지정할 수 있음 (ex. 값이 `TreeSet`인 `TreeMap` 반환)

<br>

**4️⃣ joining**

- `CharSequence` 인스턴스의 스트림에만 적용 가능

- 매개변수가 없는 경우, 단순히 원소들을 연결하는 수집기 반환

- 매개변수 하나인 경우, `CharSequence` 타입의 **구분문자**를 매개변수로 받아 연결 부위에 이 구분문자를 삽입

  >구분문자로(,) 입력하면 CSV 형태의 문자열 생성

- 매개변수 세개인 경우, **구분문자**, **접두문자(prefix)**, **접미문자(suffix)**

  > [ , ] 로 지정하면 [came, saw, conquered] 처럼 컬렉션을 출력한 듯한 문자열 생성

<br>

----

<br>

### 📌 핵심정리

**스트림 파이프라인 프로그래밍의 핵심은 부작용 없는 함수 객체에 있다**

**스트림뿐 아니라 스트림 관련 객체에 건네지는 모든 함수 객체가 부작용이 없어야 한다**

**종단 연산 중 forEach는 스트림 수행한 계산 결과를 보고할 때만 이용해야 한다**

**계산 자체에는 이용하지 말자**

**스트림을 올바로 사용하려면 수집기를 잘 알아둬야 한다**

**가장 중요한 수집기 팩터리는 toList, toSet, toMap, groupingBy, joining 이다**
