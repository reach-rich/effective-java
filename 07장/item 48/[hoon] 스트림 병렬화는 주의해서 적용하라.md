## 1. 들어가기

Java 8에서는 Stream API를 지원함과 동시에 parallel 메서드를 통한 Stream의 병렬 실행도 지원합니다.

하지만, 병렬 실행 메서드를 호출한다고 해서 무조건 올바르게 작성하는 것은 아닙니다.

그럼 어떻게 병렬 실행 메서드를 사용해야 할까요?

## 2. 병렬 실행 메서드를 잘못 사용한 예시

Item 45에서 다루었던 메르센 소수 프로그램을 보면 다음과 같습니다.

```java
/* 메르센 소수 예시 */
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

해당 코드에서 속도를 높이기 위해 `parallel()`을 호출하면 어떻게 될까요?

안타깝게도 아무것도 출력하지 못하면서 CPU를 잡아먹는 상태가 무한히 계속됩니다.

> 아니... Stream의 병렬 실행을 지원한다면서?

해당 코드의 문제는 두 가지입니다.

1. 데이터 소스가 Stream.iterate이거나 중간 연산으로 limit를 쓰면 성능 개선을 할 수 없다.

2. Pipeline 병렬화는 limit를 다룰 때 CPU 코어가 남는다면 원소를 몇 개 더 처리한 후

   제한된 개수 이후의 결과를 버려도 아무런 해가 없다고 가정한다.

   ✏️ 원소 하나를 계산하는 비용이 이전 원소 전부를 계산한 비용을 합친 것보다 많이 든다.

   <br>


그래서 이 Pipeline은 자동 병렬화 알고리즘이 제 기능을 못하게 마비시킵니다.

그러므로 Stream Pipeline을 함부로 병렬화하면 오히려 성능이 나빠질 수도 있습니다.

그럼 어떻게 사용해야 병렬화를 할 수 있는걸까요?

## 3. Stream을 병렬화하기 위한 조건

대체로 Stream 소스가 `ArrayList`, `HashMap`, `HashSet`, `ConcurrentHashMap`의 인스턴스이거나

배열, int 범위, long 범위일 때 가장 병렬화의 효과가 좋습니다.

이 자료구조들의 특징은 다음과 같습니다.

1. 데이터를 원하는 크기로 정확하게 나눌 수 있어 Thread에 분배하기 좋은 특징이 있다.

2. 원소들을 순차적으로 실행할 때의 참조 지역성이 뛰어난 특징이 있다.

   > 참조 지역성 (Locality of Reference)
   >
   > 동일한 값 또는 해당 값에 관계된 스토리지 위치가 자주 액세스되는 특성으로
   >
   > 이웃한 원소의 참조들이 메모리에 연속해서 저장되어 있을수록 참조 지역성이 좋다.

   참조 지역성이 낮으면 Thread는 데이터가 전송되어 오기를 하염없이 기다립니다.

   따라서 참조 지역성은 병렬화 시에 중요한 요소로 작용합니다.

   <br>

그 외에 Stream Pipeline의 종단 연산 동작 방식 역시 병렬 수행 효율에 영향을 줍니다.

종단 연산이 만약, 컬렉션들을 합치는 collect 메서드라면 병렬화에 적합하지 않습니다.

하지만, Stream Pipeline에서 만들어진 모든 원소를 하나로 합치는 `reduce` 메서드 중 하나,

또는 `min`, `max`, `count`, `sum`과 같이 완성된 형태로 제공되는 메서드는 병렬화에 적합합니다.

## 4. Stream 병렬화를 잘 사용한 예시

그럼 병렬화를 적용한 예시를 살펴봅시다.

```java
/* 소수 계산 스트림 파이프라인 */
   static long pi(long n) {
      return LongStream.rangeClosed(2, n)
                       .parallel()
                       .mapToObj(BigInteger::valueOf)
                       .filter(i -> i.isProbablePrime(50))
                       .count();
   }
```

해당 코드는 n보다 작거나 같은 소수의 개수를 계산하는 함수입니다.

`parallel()` 호출을 하나 추가한 것인데 글쓴이의 컴퓨터에서는 3.37배 빨라졌다고 합니다.

만약, n이 크다면 이 방식으로 계산하는 것 대신 레머의 공식을 사용해봅시다.

그리고 무작위 수들로 이뤄진 Stream을 병렬화하려거든 SplittableRandom 인스턴스를 이용합시다.

## 5. Stream 병렬화 시 주의사항

이번에는 Stream 병렬화 시 주의사항을 알아봅시다.

1. 병렬화 시 안전 실패가 발생하지 않도록 주의한다.

   안전 실패란, 병렬화한 Pipeline이 사용하는 filters 등의 함수 객체가 명세대로 동작하지 않는 것으로

   예를 들어 reduce 연산의 누적기와 결합기 함수는 
   
   반드시 결합법칙을 만족하고, 간섭받지 않으며, 상태를 갖지 않아야 합니다.

   <br>

2. 병렬화는 순서를 보장하지 않는다.

   순서를 보장하고 싶을 때는 종단 연산에 `forEach` 대신 `forEachOrdered`를 사용합시다.

   <br>

3. 병렬화에 드는 추가 비용도 고려해야 한다.

   데이터 소스 Stream이 잘 나눠지고 알맞은 종단 연산과 함수 객체들 간에 서로 간섭하지 않더라도

   Pipeline이 수행하는 작업이 병렬화에 드는 비용을 상쇄하지 못한다면 성능 향상은 미미할 수 있습니다.

   이럴 때는 다음 계산을 통해 병렬화가 적합한지 고려해봅시다.
   
   > Stream 내부 원소 수 * 원소 당 수행되는 코드 줄 수

   해당 값이 수십만은 되어야 성능 향상을 볼 수 있습니다.

   <br>

4. 변경 전후로 반드시 성능을 테스트해야 한다.

   보통, 병렬 Stream Pipeline은 같은 Thread-Pool을 사용하므로

   잘못된 Pipeline 하나가 시스템의 다른 부분까지 악영향을 줄 수 있으므로 주의해야 합니다.

## 6. 정리

   이번 포스트는 Stream 병렬화에 대해 알아보았습니다.

   Stream을 잘못 병렬화하게 되면 프로그램을 오동작하게 하거나 성능을 급격하게 떨어뜨리므로

   정확하고 성능 향상이 될 것이라는 확신 없이는 Stream 병렬화를 시도하지 맙시다.

   만약, 병렬화하는 편이 낫다고 믿는다면 운영 환경과 유사한 조건에서 정확성과 성능을 체크한 후,

   확실히 나아졌다고 판단되었을 때만 병렬화 코드를 운영 코드에 반영합시다.