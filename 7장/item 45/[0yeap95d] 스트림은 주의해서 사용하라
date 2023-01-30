### 📚 스트림 API

**: 스트림 API 다량의 데이터 처리 작업을 돕고자 추가되었음!!**

<br>

**API가 제공하는 추상 개념 중 핵심은 두 가지**

1. 스트림은 데이터 원소의 유한 혹은 무한 시퀀스를 뜻한다
2. 스트림 파이프라인은 이 원소들로 수행하는 연산 단계를 표현하는 개념이다

*스트림 안의 데이터 원소들은 객체 참조나 기본 타입 값(int, long, double)이다*

<br>

---

<br>

### 🔍 스트림파이프라인

**🚩 스트림 파이프라인 순서**

1. 소스스트림 
2. 중간 연산
   - 하나 이상이 있을 수 있음
   - 모두 한 스트림을 다른 스트림으로 변환
   - 변환된 스트림 원소 타입은 변환 전 스트림의 원소 타입과 같을 수도 있고 다를 수도 있음
3. 종단 연산
   - 마지막 중간 연산이 내놓은 스트림에 최후의 연산을 가함
   - 원소를 정렬해 컬렉션에 담거나, 특정 원소 하나를 선택하거나, 모든 원소를 출력

<br>

**🚩스트림 파이프라인은 지연 평가된다**

- 평가는 종단 연산이 호출될 때 이뤄지며, 종단 연산에 쓰이지 않는 데이터 원소는 계산에 쓰이지 않음
- 지연 평가가 무한 스트림을 다룰 수 있게 해주는 열쇠
- 종단 연산이 없는 스트림 파이프라인은 아무 일도 하지 않는 명령어인 `no-op`과 같으므로, 종단 연산을 빼먹는 이리 없도록 해야함

<br>

**🚩스트림 파이프라인은 순차적으로 수행된다**

- 파이프라인을 병렬로 실행하려면 파이프라인을 구성하는 스트림 중 하나에서 `parallel` 메서드를 호출해주기만 하면 되지만, 효과를 볼 수 있는 상황이 많지 않음

<br>

---

<br>

### ❓스트림 API를 언제 써야할까

**: 어떠한 계산이라도 해낼 수 있지만, 잘못 쓰면 읽기 어렵고 유지보수도 힘들어 짐**

<br>



**✏ #01 예제소스 | 사전 하나를 훑어 원소 수가 많은 아나그램 그룹들을 출력**

*아나그램(anagram)이란 철자를 구성하는 알파벳이 같고 순서만 다른 단어를 뜻함*

```java
public class Anagrams {
	  public static void main(String[] args) throws IOException {
      File dictionary = new File(args[0]);
      int minGroupSize = Interger.parseInt(args[1]);
      
      Map<String, Set<String>> groups = new HashMap<>();
      try(Scanner s = new Scanner(dictionary)){
      	while(s.hasNext()){
        	String word = s.next();
            groups.computeIfAbsent(alphabetize(word), (unused) -> new TreeSet<>()).add(word); // 주목!!
        }
      }
      
      for(Set<String> group : groups.values())
      	if(group.size() >= minGroupSize)
        	System.out.println(group.size() + ":" + group);
    }
    
    public static String alphabetie(String s){
    	char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```

>- computeIfAbsent 메서드는 맵안에 키가 있는지 찾은 다음, 있으면 단순히 그 키에 매핑된 값을 반환
>- 키가 없으면 건네진 함수 객체를 키에 적용하여 값을 계산해낸 다음 그 키와 값을 매핑해놓고, 계산된 값을 반환
>- 해당 메서드를 이용하여 각 키에 다수의 값을 매핑하는 맵을 쉽게 구현 가능

<br>

**✏ #02 예제소스 | 스트림을 과하게 사용 - 따라 하지 말 것!**

```java
public class StreamAnagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        //사전 파일을 제대로 닫기 위해 try-with-resources 활용
        try (Stream<String> words = Files.lines(dictionary)) {
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

>- 코드는 짧지만 읽기가 어려움
>- 위 소스처럼 스트림을 과용하면 프로그램이 읽거나 유지보수하기 어려워짐

<br>

**✏ #03 예제소스 | 스트림을 적절히 활용**

```java
public class HybridAnagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(groupingBy(word -> alphabetize(word)))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .forEach(g -> System.out.println(g.size() + ": " + g));
        }
    }
    // alphabetize 메서드는 위의 코드와 동일
}
```

>- `try-with-resources` 블록에서 사전 파일을 열고, 파일의 모든 라인으로 구성된 스트림을 얻음
>- 스트림 변수의 이름을 `words`로 지어 스트림 안의 각 원소가 `word`임을 명확히 했음
>- 중간 연산이 없으며, 종단 연산에서 모든 단어를 맵으로 모음
>- 이 맵들은 단어들을 아나그램끼리 묶어놓은 것 (실질적으로앞선 두 프로그램이 생성한 맵고 같음)
>- 맵의 `values()`가 반환한 값으로부터 새로운 `Stream<List<String>> `스트림을 열어줌
>- 그 리스트들 중 원소가 `minGroupSize`보다 적은 것은 필터링 되고, 종단 연산인 `forEach`는 살아남은 리스트를 출력

<br>

---

### ❗ 자바는 char용 스트림을 지원하지 않는다 

**: 자바가 char 스트림을 지원하는 건 불가능**

```java
"Hello World!".chars().forEach(System.out::print); // 출력: 739488237102..

"Hello World!".chars().forEach(x -> System.out.println((char) x));
```

>- `"Hello word!".char()`가 반환하는 스트림의 원소는 `int` 값
>- 올바른 `print` 메서드를 호출하려면 형변환을 명시적으로 해줘야 함

<br>

**✔ char 값들을 처리할 때는 스트림을 삼가는 편이 좋음!**

<br>

---

<br>

### ⚔ 스트림 vs 코드블록

**스트림 파이프라인은 되풀이되는 계산을 함수 객체로 표현하지만, 반복 코드에서는 코드 블록을 사용해 표현**

<br>

**1️⃣ 코드 블록을 사용해야 하는 경우**

- 코드 블록에서는 범위 안의 지역변수를 읽고 수정할 수 있으나, 람다에서는 final이거나 사실상 final인 변수만 읽을 수 있고, 지역변수를 수정하는 건 불가능
- 코드 블록에서는 return 문을 사용해 메서드에서 빠져나가거나, break나 continue문으로 블록 바깥의 반복문을 종료하거나 반복을 한 번 건너뛸 수 있음
- 메서드 선언에 명시된 검사 예외를 던질 수 있음

<br>

**2️⃣ 스트림을 사용해야 하는 경우**

- 원소들의 시퀀스를 일관되게 변환
- 원소들의 시퀀스를 필터링
- 원소들의 시퀀스를 하나의 연산을 사용해 결합(더하기, 연결하기, 최소값 구하기 등)
- 원소들의 시퀀스를 컬렉션에 모음(공통 속성을 기준으로 묶어가며)
- 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾음

<br>

**3️⃣ 스트림으로 처리하기 힘든 경우**

- 한 데이터가 파이프라인의 여러 단계를 통과할 때 각 단계에서 값들에 동시 접근하기 어려운 경우

  *(스트림 파이프라인은 한 값을 다른 값에 매핑하면 원래의 값을 잃는 구조)*

- 가능한 경우라면, 앞 단계의 값이 필요할 때 매핑을 거꾸로 수행하는 방법

*ex) 메르센 소수???*

<br>

**4️⃣ 스트림과 반복 중 어느 쪽을 써야 할지 바로 알기 어려운 작업**

​	**✏ #04 예제소스 | 데카르트 곱 계산을 반복 방식으로 구현**

```java
private static List<Card> newDeck(){
    List<Card> result = new ArrayList<>();
    for(Suit suit : Suit.values())
        for(Rank rank : Rank.values())
            result.add(new Card(suit, rank));
    return result;
}
```

>`for-each` 반복문을 중첩해 구현

<br>

**✏ #05 예제소스 | 데카르트 곱 계산을 스트림 방식으로 구현**

```java
private static List<Card> newDeck(){
    return Stream.of(Suit.values())
    .flatMap(suit ->
        Stream.of(Rank.values())
            .map(rank -> new Card(suit, rank)))
            .collect(toList());
}
```

>- 중첩된 람다를 사용해 구현
>- 중간 연산으로 사용한 flatMap은 스트림의 원소 각각을 하나의 스트림으로 매핑한 다음, 그 스트림들을 다시 하나의 스트림으로 합침, 이를 **평탄화**라고 함

<br>

**✔ 이러한 두 코드는 개인의 취향과 프로그래밍 환경의 문제!**

---

<br>

### 📌 핵심정리

**스트림을 사용해야 멋지게 처리할 수 있는 일이 있고, 반복 방식이 더 알맞은 일도 있다**

**그리고 수많은 작업이 이 둘을 조합했을 때 가장 멋지게 해결된다**

**어느 쪽을 선택하는 확고부동한 규칙은 없지만 참고할 만한 지침 정도는 있다**

**어느 쪽이 나은지가 확연히 드러나는 경우가 많겠지만, 아니더라도 방법은 있다**

**스트림과 반복 중 어느 쪽이 나은지 확신하기 어렵다면 둘 다 해보고 더 나은 쪽을 택해라**

<br>
