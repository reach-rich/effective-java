# 스트림에서는 부작용 없는 함수를 사용하라

스트림은 그저 또 하나의 API가 아닌 함수형 프로그래밍에 기초한 패러다임이다.

스트림이 제공하는 표현력, 속도, (상황에 따라) 병렬성을 얻으려면 API는 물론 패러다임까지 함께 받아들여야 한다.


#
### 1. 스트림에서 부작용 없는 함수 사용하기
- 스트림 패러다임의 핵심은 계산을 일렬의 변환으로 재구성하는 부분
- 각 변환 단계는 가능한 한 이전 단계의 결과를 받아 처리하는 순수 함수여야함

#### 💡 순수 함수
- 오직 입력만이 결과에 영향을 주는 함수
- 다른 가변 상태를 참조하지 않고 함수 스스로 다른 상태를 변경하지 않음


<br>

> __스트림 변환 단계에서 순수 함수를 사용하기 위해서는 스트림 연산에 건네는 함수 객체는 모두 부작용(side effect)이 없어야한다!__


#
### 2. 예시

#### 1) 스트림 패러다임을 이해하지 못하고 API만 사용

```java
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
    words.forEach(word -> {
        freq.merge(word.toLowerCase(), 1L, Long::sum);
    });
}
```
- 스트림, 람다, 메서드 참조를 사용했고 결과도 올바르지만 스트림 코드를 가장한 반복적 코드
- 스트림 API의 이점을 살리지 못해 길고 읽기 어렵고 유지보수에도 좋지 않을 뿐임
- `forEach`가 스트림 수행 연산 결과를 보여주는 거 이상의 일을 하는거에서 잘못됐음

#### 2) 스트림을 제대로 활용해 빈도표를 초기화

```java
Map<String, Long> freq;
try (Stream<String> words = new Scanner(file).tokens()) {
    freq = words.collect(groupingBy(String::toLowerCase, counting()));
}
```
- 1)과 같은 일을 하지만 스트림 API를 제대로 활용해 짧고 명확
- forEach 종단 연산을 for-each문을 생각하며 사용하면 안됨
- forEach는 대놓고 반복적이기 때문에 병렬화를 할 수도 없음

> __forEach 연산은 스트림 계산 결과를 보고할 때만 사용하고 계산할 때는 사용하지 말자__


#
### 3. `java.util.stream.Collectors`

위의 예시에서 수집기(collector)를 사용하는데 스트림을 사용하려면 꼭 배워야하는 개념
