Effective Java의  45번째 아이템 "스트림은 주의해서 사용하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 1. 스트림 API

스트림 API는 다량의 데이터 처리 작업을 돕고자 자바 8에 추가되었으며, 메서드 연쇄를 지원하는 플루언트 API(fluent API)다. 즉, 파이프라인 하나를 구성하는 모든 호출을 연결하여 단 하나의 표현식으로 완성할 수 있다. 

이 API가 제공하는 추상 개념 중 핵심은 두 가지다.

* **스트림(Stream)** : 데이터 원소의 유한 혹은 무한 시퀀스(sequence)
* **스트림 파이프라인(Stream Pipeline)** : 스트림의 원소들로 수행하는 연산 단계를 표현하는 개념

스트림 안의 데이터 원소들은 객체 참조나 기본 타입 값이다. 기본 타입 값으로는 int, long, double 이렇게 세 가지를 지원한다.

<br>

### 스트림 파이프라인

스트림 파이프라인은 소스 스트림에서 시작해 **종단 연산**으로 끝나며, 그 사이에 하나 이상의 **중간 연산**이 있을 수 있다. 

* 각 중간 연산은 스트림을 어떠한 방식으로 변환한다. 
  * 예를 들어 각 원소에 함수를 적용하거나 특정 조건을 만족 못하는 원소를 걸러낼 수 있다. 
* 중간 연산들은 모두 한 스트림을 다른 스트림으로 변환하는데, 변환된 스트림의 원소 타입은 변환 전 스트림의 원소 타입과 같을 수도 있고 다를 수도 있다.
* 종단 연산은 마지막 중간 연산이 내놓은 스트림에 최후의 연산을 가한다.
  * 예를 들어 원소를 정렬해 컬렉션에 담거나, 특정 원소 하나를 선택하거나, 모든 원소를 출력한다.

스트림 파이프라인은 **지연 평가(lazy evaluation)**된다. 평가는 종단 연산이 호출될 때 이뤄지며, 종단 연산에 쓰이지 않는 데이터 원소는 계산에 쓰이지 않는다. 이러한 지연 평가가 무한 스트림을 다룰 수 있게 해주는 열쇠다. 

기본적으로 스트림 파이프라인은 **순차적으로 수행**된다. 파이프라인을 병렬로 실행하려면 파이프라인을 구성하는 스트림 중 하나에서 parallel 메서드를 호출해주기만 하면 된다. 그러나 효과를 볼 수 있는 상황은 많지 않다.

<br>

## 2. 스트림의 적절한 활용

스트림 API는 다재다능하여 사실상 어떠한 계산이라도 해낼 수 있다. 하지만 해야만 하는 것은 아니다. **스트림을 제대로 사용하면 프로그램이 짧고 깔끔해지지만, 잘못 사용하면 읽기 어렵고 유지보수도 힘들어진다.**

다음 코드를 보자. 이 프로그램은 사전 파일에서 단어를 읽어 사용자가 지정한 문턱값보다 원소 수가 많은 아나그램 그룹을 출력한다. 아나그램이란 철자를 구성하는 알파벳이 같고 순서만 다른 단어를 말한다.

```java
public class Anagrams {
  public static void main(String[] args) throws IOException {
    File dictionary = new File(args[0]);
    int minGroupSize = Integer.parseInt(args[1]);
    
    Map<String, Set<String>> groups = new HashMap<>();
    try (Scanner s = new Scanner(dictionary)) {
      while (s.hasNext()) {
        String word = s.next();
        groups.computeIfAbsent(alphabetize(word), (unused) -> new TreeSet<>()).add(word);
      }
    }
    
    for (Set<String> group : groups.values())
      if (group.size() >= minGroupSize)
        System.out.println(group.size() + ": " + group);
  }
  
  private static String alphabetize(String s) {
    char[] a = s.toCharArray();
    Arrays.sort(a);
    return new String(a);
  }
}
```

<br>

이제 다음 코드를 살펴보자. 앞의 코드와 같은 일을 하지만 스트림을 과하게 활용한다. 사전 파일을 여는 부분만 제외하면 프로그램 전체가 단 하나의 표현식으로 처리된다. 

```java
public class Anagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(
                            groupingBy(word -> word.chars().sorted()
                                    .collect(StringBuilder::new,
                                            (sb, c) -> sb.append((char) c),
                                            StringBuilder::append
                                    ).toString()))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .map(group -> group.size() + ": " + group)
                    .forEach(System.out::println);
        }
    }
}
```

이 코드는 확실히 짧지만 읽기는 어렵다. 이처럼 **스트림을 과용하면 프로그램이 읽거나 유지보수하기 어려워진다.**

<br>

다행히 절충 지점이 있다. 다음 프로그램도 앞서의 두 프로그램과 기능은 같지만 스트림을 적당히 사용했다. 그 결과 원래 코드보다 짧을 뿐 아니라 명확하기까지 하다.

```java
public class Anagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(groupingBy(word -> alphabetize(word)))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .forEach(group -> System.out.println(group.size() + ": " + group);
        }
    }
  
  // alphabetize 메서드는 위의 코드와 같다. 
}
```

alphabetize 메서드도 스트림을 사용해 다르게 구현할 수 있다. 하지만 그렇게 하면 명확성이 떨어지고 잘못 구현할 가능성이 커진다. 심지어 느려질 수도 있다. 자바가 기본 타입은 char용 스트림을 지원하지 않기 때문이다. **char 값들을 처리할 때는 스트림을 삼가는 편이 낫다.**

<br>

## 3. 반복문 vs 스트림

스트림을 처음 쓰기 시작하면 모든 반복문을 스트림으로 바꾸고 싶은 유혹이 일겠지만, 서두르지 않는 게 좋다. 스트림으로 바꾸는게 가능할지라도 코드 가독성과 유지보수 측면에서는 손해를 볼 수 있기 때문이다. 그러니 **기존 코드는 스트림을 사용하도록 리팩터링하되, 새 코드가 더 나아 보일 때만 반영하자.**

<br>

스트림 파이프러안은 되풀이되는 계산을 함수 객체로 표현한다. 반면 반복 코드에서는 코드 블록을 사용해 표현한다. 그런데 함수 객체로는 할 수 없지만 코드 블록으로는 할 수 있는 일들이 있다.

* 코드 블록에서는 범위 안의 지역변수를 읽고 수정할 수 있다. 하지만 람다에서는 final이거나 사실상 final인 변수만 읽을 수 있고, 지역변수를 수정하는 건 불가능하다.
* 코드 블록에서는 return 문을 사용해 메서드에서 빠져나가거나, break나 continue 문으로 블록 바깥의 반복문을 종료하거나 반복을 한 번 건너뛸 수 있다. 또한 메서드 선언에 명시된 검사 예외를 던질 수 있다. 하지만 람다로는 이 중 어떤 것도 할 수 없다.

계산 로직에서 이상의 일들을 수행해야 한다면 스트림과는 맞지 않는 것이다. 반대로 다음 일들에는 스트림이 아주 안성맞춤이다.

* 원소들의 시퀀스를 일관되게 변환한다.
* 원소들의 시퀀스를 필터링한다.
* 원소들의 시퀀스를 하나의 연산을 사용해 결합한다.
* 원소들의 시퀀스를 컬렉션에 모은다.
* 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾는다.

<br>

## 4. 스트림의 한계

스트림으로 처리하기 어려운 일도 있다. 대표적인 예로, 한 데이터가 파이프라인의 여러 단계를 통과할 때 이 데이터의 각 단계에서의 값들에 동시에 접근하기는 어려운 경우다. 스트림 파이프라인은 일단 한 값을 다른 값에 매핑하고 나면 원래의 값은 잃는 구조이기 때문이다.

원래 값과 새로운 값의 쌍을 저장하는 객체를 사용해 매핑하는 우회 방법도 있지만, 그리 만족스러운 해법은 아닐 것이다. 가능한 경우라면, 앞 단계의 값이 필요할 때 매핑을 거꾸로 수행하는 방법이 나을 것이다.

예를 들어 처음 20개의 메르센 소수(Mersenne prime)를 출력하는 프로그램을 작성해보자. 메르센 수는 2^p - 1 형태의 수다. 여기서 p가 소수이면 해당 메르센 수도 소수일 수 있는데, 이 때의 수를 메르센 소수라 한다.

```java
static Stream<BigInteger> pimes() {
  return Stream.iterate(TWO, BigInteger::nextProbablePrime);
}

public static void main(String[] args) {
  primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
    .filter(mersenne -> mersenne.isProbablePrime(50))
    .limit(20)
    .forEach(System.out::println);
}
```

Stream.iterate라는 정적 팩터리는 매개변수를 2개 받는다. 첫 번째 매개변수는 스트림의 첫 번째 원소이고, 두 번째 매개변수는 스트림에서 다음 원소를 생성해주는 함수다. (무한) 스트림을 반환한다.

소수들을 사용해 메르센 수를 계산하고, 결괏값이 소수인 경우만 남긴 다음, 결과 스트림의 원소 수를 20개로 제한해놓고, 작업이 끝나면 결과를 출력한다.

<br>

이제 우리가 각 메르센 소수의 앞에 지수(p)를 출력하길 원한다고 해보자. 이 값은 초기 스트림에만 나타나므로 결과를 출력하는 종단 연산에서는 접근할 수 없다. 하지만 다행히 첫 번째 중간 연산에서 수행한 매핑을 거꾸로 수행해 메르센 수의 지수를 쉽게 계산해낼 수 있다.

지수는 단순히 숫자를 이진수로 표현한 다음 몇 비트인지를 세면 나오므로, 종단 연산을 다음처럼 작성하면 원하는 결과를 얻을 수 있다.

```java
.forEach(mp -> System.out.println(mp.bitLength() + ": " + map));
```

<br>

## 5. 핵심 정리

* 스트림을 사용해야 멋지게 처리할 수 있는 일이 있고, 반복 방식이 더 알맞는 일도 있다. 그리고 수많은 작업이 이 둘을 조합했을 때 가장 멋지게 해결된다.
* 어느 쪽이 나은지가 확연히 드러나는 경우가 많다. 만약 스트림과 반복 중 어느 쪽이 나은지 확신하기 어렵다면 둘 다 해보고 더 나은 쪽을 택하라.

<br>

## 6. Related Posts

* 병렬 스트림 (Item 48)
