Effective Java의  46번째 아이템 "스트림에서는 부작용 없는 함수를 사용하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 1. 스트림 패러다임

스트림을 처음 봐서는 이해하기 어려울 수 있다. 스트림은 그저 또 하나의 API가 아닌, 함수형 프로그래밍에 기초한 패러다임이기 때문이다. 스트림이 제공하는 표현력, 속도, 병렬성을 얻으려면 API는 말할 것도 없고 이 패러다임까지 함께 받아들여야 한다.

스트림 패러다임의 핵심은 계산을 일련의 변환으로 재구성하는 부분이다. 이때 각 변환 단계는 가능한 한 이전 단계의 결과를 받아 처리하는 **순수 함수**여야 한다. 여기서 순수 함수란 오직 입력만이 결과에 영향을 주는 함수를 말한다. 즉, 다른 가변 상태를 참조하지 않고 함수 스스로도 다른 상태를 변경하지 않는다. 이렇게 하려면 **스트림 연산에 건네는 함수 객체는 모두 side effect가 없어야 한다.** 

<br>

### 1) 스트림 패러다임을 이해하지 못한 예

다음은 텍스트 파일에서 단어별 수를 세어 빈도표로 만드는 스트림 코드다. 

```java
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
  words.forEach(word -> {
    freq.merge(word.toLowerCase(), 1L, Long::sum);
  });
}
```

스트림, 람다, 메서드 참조를 사용했고, 결과도 올바르다. 하지만 절대 스트림 코드라 할 수 없다. 스트림 코드를 가장한 반복적 코드다. 스트림 API의 이점을 살리지 못하여 같은 기능의 반복적 코드보다 길고, 읽기 어렵고, 유지보수에도 좋지 않다. 

이 코드의 모든 작업이 종단 연산인 forEach에서 일어나는데, 이때 외부 상태(빈도표)를 수정하는 람다를 실행하면서 문제가 생긴다.  

<br>

### 2) 스트림을 제대로 활용한 예

앞서 소개한 코드의 올바른 예를 살펴보자.

```java
Map<String, Long> freq;
try (Stream<String> words = new Scanner(file).token()) {
  freq = words.collect(groupingBy(String::toLowerCase, counting()));
}
```

이번엔 스트림 API를 제대로 활용해 짧고 명확하다. forEach 연산은 종단 연산 중 기능이 가장 적고 가장 덜 스트림답다. 대놓고 반복적이라서 병렬화할 수도 없다. forEach 연산은 스트림 계산 결과를 보고할 때만 사용하고, 계산하는 데는 쓰지 말자. 

이 코드는 수집기(collector)를 사용하는데, 스트림을 사용하려면 꼭 배워야하는 새로운 개념이다. 아래에서 살펴보자.

<br>

## 2. 수집기(Collector)

java.util.stream.Collectors 클래스는 메서드를 39개 가지고 있다. 다행히 복잡한 세부 내용을 잘 몰라도 이 API의 장점을 대부분 활용할 수 있다. 

수집기(Collector)는 스트림의 원소들을 객체 하나에 취합하는 전략을 캡슐화한 객체다. 수집기를 사용하면 스트림의 원소를 손쉽게 컬렉션으로 모을 수 있다.

다음은 Collectors 클래스의 메서드들이다. 

* ToList, toSet, toCollection : 리스트, 집합, 프로그래머가 지정한 컬렉션을 생성하여 반환한다.
* toMap, toConcurrentMap : 맵을 생성하여 반환한다. 
* groupingBy,  groupingByConcurrent : 분류 함수를 받고 출력으로는 원소들을 카테고리별로 모아 놓은 맵을 담은 수집기를 반환한다. 
* partitioningBy : Predicate를 받고 키가 Boolean인 맵을 반환한다. 
* counting : 원소의 수를 세는 수집기를 반환한다. 
* minBy, maxBy : 인수로 받은 비교자를 이용해 스트림에서 값이 가장 작은 혹은 가장 큰 원소를 찾아 반환한다.
* joining : 원소들을 연결하는 수집기를 반환한다.
* etc
  * summing, averaging, summarizing
  * reducing, filtering, mapping, flatMapping, collectingAndThen

<br>

## 3. 핵심 정리

* 스트림 파이프라인 프로그래밍의 핵심은 side effect 없는 함수 객체에 있다. 스트림뿐 아니라 스트림 관련 객체에 건네지는 모든 함수 객체는 side effect가 없어야 한다. 
* 종단 연산 중 forEach는 스트림이 수행한 계산 결과를 보고할 때만 이용해야 한다. 계산 자체에는 이용하지 말자.
* 스트림을 올바르게 사용하려면 수집기를 잘 알아둬야 한다.

<br>

## 4. Related Posts

* 비교자 생성 메서드 (Item 14)

