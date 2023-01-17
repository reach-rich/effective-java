## 1. 들어가기

Java 8 이전에는 다량의 데이터 처리 작업은 반복문을 통해서만 가능했습니다.

하지만, Java 8부터 다량의 데이터 처리 작업을 돕기 위해 Stream API가 도입되었습니다.

Stream API가 무엇일까요? 

## 2. Stream API

Stream API는 Stream과 Stream Pipeline라는 두 가지의 핵심 추상 개념을 제공합니다.

* Stream

   * 데이터 원소의 유한 혹은 무한 시퀀스

   * 스트림 원소로 컬렉션, 배열, 파일, 정규표현식 패턴 매처, 난수 생성기 혹은 다른 스트림이 있음

   <br>

* Stream Pipeline

   * Stream으로 수행하는 연산 단계를 표현하는 개념

   * 중간 연산: 스트림을 어떠한 방식으로 변환

   * 종단 연산: 마지막 중간 연산이 내놓은 스트림에 최후의 연산을 가함

   * 지연 평가(Lazy Evaluation) → 무한 스트림 가능

   <br>

Stream API는 Builder와 같이 메서드 연쇄를 지원하는 Fluent API입니다.

즉, Pipeline 하나를 구성하는 모든 호출을 연결해 단 하나의 표현식으로 완성할 수 있는거죠.

또한, Stream API는 다재다능해서 사실상 어떠한 계산도 할 수 있습니다.

하지만 할 수 있다는 뜻이지 해야한다는 뜻이 아니기 때문에 조심해서 사용해야 합니다.

이 Stream API를 제대로 사용하면 프로그램이 짧고 깔끔해진다는 장점이 있으나,

잘못 사용하면 가독성이 좋지 않고 유지보수가 어렵다는 단점이 있기 때문입니다.

그래서 이번에는 언제 Stream API를 사용해야 하는지 알아보겠습니다.

## 3. Stream API 사용처

먼저, Stream API를 잘못 사용한 예를 보겠습니다.

```java
   public clas Anagrams {
      public static void main(String[] args) throws IOException {
         Path dictionary = Paths.get(args[0]);
         int minGroupSize = Integer.parseInt(args[1]);
        
         try(Stream<String> words = Files.lines(dictionary)) {
            words.collect(
                groupingBy(word -> word.chars().sorted()
                           .collect(StringBuilder::new,
                                 (sb, c) -> sb.append((char) c),
                                 StringBuilder::append).toString()))
                           .values().stream()
                           .filter(group -> group.size() >= minGroupSize)
                           .map(group -> group.size() + ": " + group)
                           .forEach(System.out::println);
         }
      } 
   } 
```

한눈에 봐도 코드를 이해하기 어려운 것을 볼 수 있습니다.

이처럼 Stream API를 과용하면 프로그램의 가독성이 좋지 않고 유지보수가 어려워지는 단점이 있습니다.

이번에는 Stream API를 적절하게 사용한 예시를 보겠습니다.

```java
   public class Anagrams {
      public static void main(String[] args) throws IOException {
         Path dictionaryt = Paths.get(args[0]);
         int minGroupSize = Integer.parseInt(args[1]);
        
         try(Stream<String> words = Files.lines(dictionary)) {
            words.collect(groupingBy(word -> alphabetize(word))
                 .values().stream()
                 .filter(group -> group.size() >= minGroupSize)
                 .forEach(group -> System.out.println(group.size() + ": " + group));
         }
      }

      private static String alphbetize(String s) {
         char[] a = s.toCharArray();
         Arrays.sort(a);
         return new String(a);
      }
   }
```

확실히 앞의 예시보다 이해하기 쉬워보입니다.

정리하면, 기존 코드는 Stream을 사용하도록 리팩터링하되, 새 코드가 더 나아 보일때만 반영합시다.

다음은 Stream 사용에 안성맞춤인 경우로 이 중 하나를 수행하는 로직이라면 Stream 적용을 고려합시다.

* 원소들의 시퀀스를 일관되게 변환한다.

* 원소들의 시퀀스를 필터링한다.

* 원소들의 시퀀스를 하나의 연산(더하기, 연결하기, 최솟값 구하기)을 사용해 결합한다.

* 원소들의 시퀀스를 컬렉션에 모은다.

* 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾는다.

## 4. Stream API로 처리하기 어려운 일

만능인 것처럼 보이는 Stream API도 처리하기 어려운 일이 있습니다.

Stream Pipeline은 한 값을 매핑하고 나면 원래 값은 잃는 구조이기 때문에

pipeline의 여러 단계를 통과할 때 이 데이터의 각 단계에서 값을 동시에 접근하기 어렵습니다.

이를 해결하기 위해 기존 값과 새로운 값의 쌍을 저장하는 객체로 매핑하는 방법을 사용할 수 있지만,

코드 양이 많아져 지저분해져서 Stream API를 사용하는 목적과 멀어지게 됩니다.

그래서 어떤 특별한 경우, 매핑을 거꾸로 수행하는 방법을 사용할 수 있습니다.

예시를 보겠습니다.

```java
   static Stream<BigInteger> primes() {
      return Stream.iterate(TWO, BigInteger::nextProbablePrime);
   }
```

위의 코드는 소수인 무한 스트림을 반환하는 메서드입니다.

이 코드를 통해 우리는 메르센 소수를 출력해볼 것입니다.

여기서 메르센 수란, 2<sup>p</sup> - 1 형태의 수로, 

p가 소수이면 해당 메르센 수도 소수일 수 있는데 이 때의 수를 **메르센 소수**라고 합니다.

그래서 메르센 소수를 출력하는 프로그램은 다음과 같습니다.

```java
   public static void main(String[] args) {
      primes().map(p -> TWO.pow(p.intValueExact()))
              .filter(mersenne -> mersenne.isProbablePrime(50))
              .limit(20)
              .forEach(System.out::println);
   }
```

여기서 각 메르센 소수의 앞에 지수(p)를 출력하기 원한다고 가정해봅시다.

이 값은 초기 스트림에만 나타나기 때문에 결과를 출력하는 종단 연산에서는 접근할 수 없습니다.

하지만, 지수는 단순히 숫자를 이진수로 표현한 다음, 몇 비트인지 알 수 있기 때문에

첫 번째 중간 연산에서 수행한 매핑을 거꾸로 수행해서 쉽게 계산할 수 있습니다.

```java
   public static void main(String[] args) {
      primes().map(p -> TWO.pow(p.intValueExact()))
              .filter(mersenne -> mersenne.isProbablePrime(50))
              .limit(20)
              .forEach(mp -> System.out.println(mp.bitLength() + ": " + mp));
    }
```

## 5. Stream API vs 반복문

Stream과 반복문 중 어느 쪽을 써야 할지 바로 알기 어려운 작업도 있습니다.

카드 덱을 초기화 하는 예시를 보겠습니다.

```java
   private static List<Card> newDeck() {
      List<Card> result = new ArrayList<>();
      for(Suit suit : Suit.values()) {
         for(Rank rank : Rank.values()) {
            result.add(new Card(suit, rank));
         }
      }

      return result;
   }
```

해당 예시는 일반 반복문으로 만든 숫자(rank)와 무늬(suit)가 다른 모든 카드를 만드는 코드입니다. 

이를 Stream을 사용해 변경하면 다음과 같습니다.

```java
   private static List<Card> newDeck() {
      return Stream.of(Suit.values())
                   .flatMap(suit -> Stream.of(Rank.values())
                                          .map(rank -> new Card(suit, rank)))
                   .collect(toList());
   }
```

프로그래머마다 편하게 생각하는 코드가 있을 것입니다.

만약, Stream을 사용한 코드가 나아 보이고 동료들도 Stream을 잘 사용한다면 Stream을 사용하되,

그 이외의 경우에는 반복문을 사용합시다.

## 6. 정리

이번 포스트는 Stream API에 대해 알아보았습니다.

Stream API는 제대로 사용하면 프로그램이 짧고 깔끔해진다는 장점이 있으나,

잘못 사용하면 가독성이 좋지 않고 유지보수가 어렵다는 단점을 가지고 있기 때문에 주의해야 합니다.

그리고 Stream을 사용해야 멋지게 처리할 수 있는 일이 있고,

반복 방식을 사용해야 멋지게 처리할 수 있는 일이 있으므로 둘다 해보고 더 나은 쪽을 선택합시다.