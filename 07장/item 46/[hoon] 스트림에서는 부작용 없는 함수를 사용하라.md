## 1. 들어가기

Java 8부터 도입된 Stream API는 함수형 프로그래밍에 기초한 패러다임이기 때문에

처음 봐서는 이해하기 어렵고 Stream Pipeline으로 원하는 작업을 표현하는 것조차 어려울 수 있습니다.

Stream 패러다임의 핵심은 계산을 일련의 변환으로 재구성 하는 부분인데

이때 각 변환 단계는 가능한 한 이전 단계의 결과를 받아 처리할 수 있어야 합니다.

그럼, Stream을 어떻게 사용해야 하는지 알아봅시다.

## 2. Stream을 잘못 사용한 예

```java
   Map<String, Long> freq = new HashMap<>();

   try(Stream<String> words = new Scanner(file).tokens()) {
      words.forEach(word -> {
         freq.merge(word.toLowerCase(), 1L, Long::sum);
      })
   }
```

해당 예시는 Stream을 사용했고, 결과도 올바르지만 절대 Stream 코드라고 할 수 없습니다.

이는 단지, Stream 코드를 가장한 반복적 코드라고 할 수 있습니다.

그리고 같은 기능(for, while)의 반복적 코드보다 더 길고 읽기 어려우며 유지보수에도 좋지 않습니다.

그럼 Stream을 어떻게 제대로 사용할 수 있을까요?

## 3. Stream을 잘 사용한 예

```java
   Map<String, Long> freq = new HashMap<>();

   try(Stream<String> words = new Scanner(file).tokens()) {
      freq = words.collect(groupingBy(String::toLowerCase, counting()));
   }
```

앞서 Stream을 잘못 사용한 예시와 동일한 기능을 하지만 이번에는 올바르게 Stream을 사용했습니다.

자바 프로그래머라면 for-each 반복문을 사용할 줄 아는데

for-each 반복문은 Stream의 forEach 종단 연산과 비슷하게 생겨서 잘못 사용하기 쉽습니다.

하지만 Stream의 forEach 연산은 기능이 가장 적고 가장 덜 Stream 답기 때문에

Stream 계산 결과를 보고할 때만 사용하고, 계산하는 데는 사용을 지양해야 합니다.

## 4. Collector

이번에는 Stream을 사용하기 위해서는 꼭 숙지해야 하는 Collector에 대해 알아보겠습니다.

Collector는 Stream의 원소를 손쉽게 컬렉션으로 모으는 역할을 합니다.

Collector의 메서드를 무려 39개나 가지고 있고, 그 중에는 타입 매개변수가 5개인 것도 있어서

모두 외우는 것은 무리가 있으므로 간략하게 어떻게 사용하는지만 알아봅시다.

## 5. Collector의 종류

* `toList()`

   `toList`는 Stream의 원소들을 리스트로 모을 수 있습니다.

   ```java
      List<String> topTen = freq.keySet().stream()
                                         .sorted(comparing(freq::get).reversed())
                                         .limit(10)
                                         .collect(toList());
   ```

   <br>

* `toSet()`

   `toSet`은 Stream의 원소들을 집합으로 모을 수 있습니다.

   ```java
      Set<String> topTen = freq.keySet().stream()
                                        .sorted(comparing(freq::get).reversed())
                                        .limit(10)
                                        .collect(toSet());
   ```

   <br>

* `toCollection()`

   `toCollection`은 Stream의 원소들을 프로그래머가 지정한 컬렉션 타입으로 모을 수 있습니다.

   ```java
      Set<String> topTen = freq.keySet().stream()
                                        .sorted(comparing(freq::get).reversed())
                                        .limit(10)
                                        .collect(Collectors.toCollection(TreeSet::new));
   ```

   <br>

* `toMap()`

   `toMap`은 Stream의 원소들을 맵으로 매핑할 수 있습니다.

   `toMap`은 3가지 종류가 있습니다.

   1. `toMap(keyMapper, valueMapper)`

      Stream 원소를 키, 값에 매핑하는 함수를 인수로 받는 형태

      ```java
         Stream.of(values())
               .collect(toMap(Object::toString, e -> e));
      ```

       <br>

   2. `toMap(keyMapper, valueMapper, mergeFunction)`

      Stream 원소를 키, 값에 매핑하는 함수, 그리고 충돌 시 병합 함수를 인수로 받는 형태

      ```java
         Stream.of(values())
               .collect(toMap(Object::toString, e -> e, (oldVal, newVal) -> newVal));
      ```

       <br>

   3. `toMap(keyMapper, valueMapper, mergeFunction, mapFactory)`

      Stream 원소를 키, 값에 매핑하는 함수, 충돌 시 병합 함수 그리고 맵 팩터리를 인수로 받는 형태

      ```java
         Stream.of(values())
               .collect(toMap(Object::toString, e -> e, (oldVal, newVal) -> newVal, TreeMap::new));
      ```

      <br>

   그리고 각 `toMap`은 병렬 실행을 할 수 있는 `toConcurrentMap`이 있습니다.

   <br>

* `groupingBy()`

   `groupingBy`는 Stream 원소들을 카테고리 별로 모아 놓은 맵을 반환합니다.

   `groupingBy` 또한 3가지 종류가 있습니다.

   1. `groupingBy(classifier)`

      분류 함수를 인수로 받는 형태

      ```java
         words.collect(groupingBy(word -> alphabetize(word)));
      ```

      <br>

   2. `groupingBy(classifier, downstream)`

      분류 함수와 다운스트림을 인수로 받는 형태

      ```java
         words.collect(groupingBy(word -> alphabetize(word), toSet()));
      ```

      ```java
         words.collect(groupingBy(word -> alphabetize(word), counting()));
      ```

      <br>

   3. `groupingBy(classifier, MapFactory, downstream)`

      분류 함수, 맵 팩터리 그리고 다운스트림을 인수로 받는 형태 (점층적 인수 목록 패턴 위배)

      ```java
         words.collect(groupingBy(word -> alphabetize(word), TreeMap::new, toSet()));
      ```

      <br>

   그리고 각 `groupingBy`는 병렬 실행을 할 수 있는 `groupingByConcurrent`도 있습니다.

   또한, 많이 쓰이진 않지만 `groupingBy`의 사촌 격인 `partitioningBy`도 있습니다.

   <br>

* 다운 스트림 전용 Collector

   앞서 보았던 갯수를 세는 `counting`을 포함하여
   
   다운 스트림에서 사용할 목적으로 만들어진 메서드가 16개 있습니다.

   <br>

* 특이한 케이스의 Collector

   이번에 볼 Collector는 Collectors에 정의되어 있지만 수집과 관련이 없는 Collector입니다.

   1. `minBy`

      인수로 받은 비교자를 이용해 Stream에서 값이 가장 작은 원소를 찾아 반환합니다.

      ```java
         albums.collect(toMap(Album::artist, a -> a, minBy(comparing(Album::sales))));
      ```

      <br>

   2. `maxBy`

      인수로 받은 비교자를 이용해 Stream에서 값이 가장 큰 원소를 찾아 반환합니다.

      ```java
         albums.collect(toMap(Album::artist, a -> a, maxBy(comparing(Album::sales))));
      ```

      <br>

   3. `joining`

      단순히 Stream 원소들을 연결해 반환합니다.

      ```java
         words.collect(joining(","));
      ```

## 6. 정리

이번 포스트는 Stream API를 잘 활용하는 방법에 대해 알아보았습니다.

Stream Pipeline 프로그래밍 핵심은 부작용 없는 함수 객체에 있습니다.

종단 연산 중 forEach는 Stream이 수행한 계산 결과를 보고할 때만 사용해야 하고,

Stream을 컬렉션으로 반환하기 위해서는 Collector를 잘 활용해야 합니다.