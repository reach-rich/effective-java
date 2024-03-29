Effective Java의  48번째 아이템 "스트림 병렬화는 주의해서 적용하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 1. 자바의 동시성 프로그래밍

주류 언어 중, 동시성 프로그래밍 측면에서 자바는 항상 앞서갔다. 

* 릴리스(1996년) : 스레드, 동기화, wait/notify를 지원했다. 
* 자바 5 : 동시성 컬렉션인 java.util.concurrent 라이브러리와 실행자(Executor) 프레임워크를 지원했다. 
* 자바 7 : 고성능 병렬 분해 프레임워크인 포크-조인(fork-join) 패키지를 추가했다. 
* 자바 8 : parallel 메서드만 한 번 호출하면 파이프라인을 병렬 실행할 수 있는 스트림을 지원했다. 

이처럼 자바로 동시성 프로그램을 작성하기가 점점 쉬워지고는 있지만, 이를 올바르고 빠르게 작성하는 일은 여전히 어려운 작업이다. 동시성 프로그래밍을 할 때는 안정성(safety)과 응답 가능(liveness) 상태를 유지하기위해 애써야 하는데, 병렬 스트림 파이프라인 프로그래밍에서도 다를 바 없다. 

<br>

## 2. 병렬 스트림에서 주의할 점

### 1) 데이터 소스가 Stream.iterate거나 중간 연산으로 limit을 쓰면 성능 개선을 기대할 수 없다.

아이템 45에서 다루었던 메르센 소수를 생성하는 프로그램을 다시 살펴보자.

```java
public static void main(String[] args) {
  primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
    .filter(mersenne -> mersenne.isProbablePrime(50))
    .limit(20)
    .forEach(System.out::println);
}

static Stream<BigInteger> primes() {
  return Stream.iterate(TWO, BigInteger::nextProbablePrime);
}
```

이 프로그램의 속도를 높이기 위해 스트림 파이프라인 parallel()를 호출하면 어떻게 변할까? 안타깝게도 이 프로그램은 아무것도 출력하지 못하면서 CPU는 90%나 잡아먹는 상태가 무한히 계속된다.

프로그램이 느려진 원인은 스트림 라이브러리가 이 파이프라인을 병렬화하는 방법을 찾아내지 못했기 때문이다. 그리고 그 원인은  데이터 소스로 Stream.iterate를 사용하고 있고, 스트림의 중간 연산으로 limit을 사용했기 때문이다. 

<br>

대체로 스트림의 소스가 ArrayList, HashMap, HashSet, ConcurrentHashMap의 인스턴스거나 배열, int 범위, long 범위일 때 병렬화의 효과가 가장 좋다. 이 자료구조들은 모두 데이터를 원하는 크기로 정확하고 손쉽게 나눌 수 있어서 일을 다수의 스레드에 분배하기에 좋다는 특징이 있다. 

이 자료구조들의 또 다른 중요한 공통점은 원소들을 순차적으로 실행할 때의 참조 지역성이 뛰어나다는 것이다. 이웃한 원소의 참조들이 메모리에 연속해서 저장되어 있다는 뜻이다.

참조 지역성이 낮으면 스레드는 데이터가 주 메모리에서 캐시 메모리로 전송되어 오기를 기다리며 대부분 시간을 멍하니 보내게 된다. 따라서 참조 지역성은 다량의 데이터를 처리하는 벌크 연산을 병렬화할 때 아주 중요한 요소로 작용한다. 

<br>

### 2) 스트림 파이프라인의 종단 연산의 동작 방식은 병렬 수행 효율에 영향을 준다.

종단 연산에서 수행하는 작업량이 파이프라인 전체 작업에서 상당 비중을 차지하면서 순차적인 연산이라면 파이프라인 병렬 수행의 효과는 제한될 수밖에 없다. 

중단 연산 중 병렬화에 가장 적합한 것은 축소(reduction)다. 축소는 파이프라인에서 만들어진 모든 원소를 하나로 합치는 작업이다. anyMatch, allMatch, noneMatch처럼 조건에 맞으면 바로 반환되는 메서드도 병렬화에 적합하다.

반면, 가변 축소(mutable reduction)를 수행하는 Stream의 collect 메서드는 병렬화에 적합하지 않다. 컬렉션들을 합치는 부담이 크기 때문이다.

<br>

### 3) 스트림을 잘못 병렬화하면 성능이 나빠질 뿐만 아니라 결과 자체가 잘못되거나 예상 못한 동작이 발생할 수 있다.

결과가 잘못되거나 오동작하는 것은 안전 실패(safety failure)라 한다. 안전 실패는 병렬화한 파이프라인이 사용하는 mappers, filters, 혹은 프로그래머가 제공한 다른 함수 객체가 명세대로 동작하지 않을 때 벌어질 수 있다.

Stream 명세는 이때 사용되는 함수 객체에 관한 엄중한 규약을 정의해놨다. 예를 들어 Stream의 reduce 연산에 건네지는 accumulator(누적기)와 combiner(결합기) 함수는 반드시 결합법칙을 만족하고, 간섭받지 않고, 상태를 갖지 않아야 한다.

<br>

### 4) 파이프라인이 수행하는 진짜 작업이 병렬화에 드는 추가 비용을 상쇄하지 못한다면 성능 향상은 미미할 수 있다.

데이터 소스 스트림이 효율적으로 나눠지고, 병렬화하거나 빨리 끝나는 종단 연산을 사용하고, 함수 객체들도 간섭하지 않더라도 성능 향상은 미미할 수 있다. 실제로 성능이 향상될지를 추정해보는 간단한 방법이 있다. 스트림 안의 원소 수와 원소당 수행되는 코드 줄 수를 곱해보자. 이 값이 최소 수십만은 되어야 성능 향상을 맛볼 수 있다. 

<br>

### 5) 반드시 성능을 테스트하여 병렬화를 사용할 가치가 있는지를 확인해야 한다.

스트림 병렬화는 오직 성능 최적화 수단임을 기억해야 한다. 다른 최적화와 마찬가지로 변경 전후로 테스트를 해봐야 한다. 이상적으로는 운영 시스템과 흡사한 환경에서 테스트하는 것이 좋다. 보통은 병렬 스트림 파이프라인도 공통의 포크-조인 풀에서 수행되므로(즉, 같은 스레드 풀을 사용하므로) 잘못된 파이프라인 하나가 시스템의 다른 부분의 성능에까지 악영향을 줄 수 있다.

<br>

### 6) 무작위 수들로 이루어진 스트림을 병렬화하려거든 ThreadLocalRandom(혹은 Random)보다는 SplittableRandom 인스턴스를 이용하자.

SplittableRandom은 정확히 이럴 때 쓰고자 설계된 것이라 병렬화하면 성능이 선형으로 증가한다. 

한편, ThreadLocalRandom은 단일 스레드에서 쓰고자 만들어졌다. 병렬 스트림용 데이터 소스로도 사용할 수는 있지만 상대적으로 느릴 것이다. 그리고 Random은 모든 연산을 동기화하기 때문에 병렬 처리하면 최악의 성능을 보일 것이다.

<br>

## 3. 핵심 정리

* 계산도 올바로 수행하고 성능도 빨라질 거라는 확신 없이는 스트림 파이프라인 병렬화는 시도조차 하지 말라.
* 스트림을 잘못 병렬화하면 프로그램을 오동작하게 하거나 성능을 급격히 떨어뜨린다.
* 병렬화하는 편이 낫다고 믿더라도, 수정 후의 코드가 여전히 정확한지 확인하고 운영 환경과 유사한 조건에서 수행해보며 성능지표를 유심히 관찰하라.

<br>

## 4. Related Posts

* 부작용 없는 함수 (Item 46)
* 성능 최적화 (Item 67)

