# 스트림은 주의해서 사용하라

### 1. 스트림

> __스트림(stream)은 데이터 원소의 유한 혹은 무한 시퀀스(sequence)__

- 스트림의 원소들은 어디로부터든 올 수 있음
- 대표적으로 컬렉션, 배열, 파일, 정규표현식 패턴 캐처, 난수 생성기가 있음
- 스트림 안의 데이터 원소들은 객제 참조나 기본 타입의 값
- 기본 타입으로 int, long, double을 지원

<br>

> __스트림 파이프라인(stream pipeline)은 이 원소들로 수행하는 연산 단계를 표현하는 개념__

- 소스 스트림에서 시작해 종단 연산으로 끝나며 그 사이에 하나 이상의 중간 연산이 있을 수 있음
- 각 중간 연산은 스트림을 어떠한 방식으로 변환
- 각 원소에 함수를 적용하거나 특정 조건을 만족 못하는 원소를 걸러낼 수 있음
- 중간 연산은 한 스트림을 다른 스트림으로 변환하고 변환 전 스트림과 다를 수 있음
- 종단 연산은 마지막 중간 연산이 내놓은 스트림에 최후의 연산 추가 (정력, 특정원소 선택, 모든 원소 출력)
- 스트림 파이프라인은 지연평가되며 이는 무한 스트림을 다룰 수 있게 해줌


<br>

> 스트림 API는 메서드 연괘를 지원하는 플루언트 API

- 다량의 데이터 처리 작업(순차와 병렬 모두!)을 돕고자 자바 8에 추가
- 파이프라인 하나를 구성하는 모든 호출을 연결해 단 하나의 표현식으로 완성
- 파이프라인 여러개를 연결해 표현식 하나로 만들기도 가능
- 스트림 파이프라인은 순차적으로 실행되면 병렬으로 실행 시 `paeallel` 메서드 호출하면 됨 (효과를 보기 쉽진 않음)

#
### 2. 스트림을 적절하게 
스트림 API는 기능이 많아 다양하게 사용할 수 있지만 남용해서는 안된다.

스트림을 제대로 사용하면 프로그램이 짧고 간결해지지만, 잘못 사용하면 읽기 어렵고 유지보수가 어려워진다.

__(1) 원소 수가 많은 아니그램(철자순서바꾸면 똑같은 친구들) 그룹 출력하는 코드__
```java
public class Anagrams {
	  public static void main(String[] args) throws IOException {
      File dictionary = new File(args[0]);
      int minGroupSize = Interger.parseInt(args[1]);
      
      Map<String, Set<String>> groups = new HashMap<>();
      try(Scanner s = new Scanner(dictionary)){
      	while(s.hasNext()){
        	String word = s.next();
            groups.computeIfAbsent(alphabetize(word), (unused) -> new TreeSet<>()).add(word);
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
- `groups.computeIfAbsent(alphabetize(word), (unused) -> new TreeSet<>()).add(word);` : 자바 8에서 추가된 computeIfAbsent 메서드 사용
- 맵 안에 키가 있는지 찾은 다음 있으면 매핑되는 값을 반환, 없으면 키와 값을 매핑하고 계산된 값을 반환

<br>

__(2) 스트림을 과하게 사용하는 코드__
```java
public class Anagrams {
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

<br>

__(3) 스트림을 적절히 사용한 코드__

```java
public class Anagrams {
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
    // 위와 동일
}
```

<br>
추가적으로 스트림은 char 타입을 지원하지 않기 때문에 int 스트림으로 반환하게 된다.

char 값을 사용할 때는 스트림을 사용하지 않는 것이 좋다.


#
### 3. 스트림을 적절하게 사용하자

무조건 스트림으로 바꾸는게 좋은 것은 아니기 떄문에 __기존 코드는 스트림을 사용해 리팩터링 하되, 새코드가 더 나아 보일 때만 반영하자!__

스트림은 되풀이 되는 계산을 함수 객체(람다, 메서드 참조)로 표현할 수 있고, 코드 블록을 사용해서 표현할 수도 있따.

<br>

__❤️‍🩹 함수 객체로는 할 수 없지만 코드 블록으로 할 수 있는 예시__

- 코드 블록에서는 범위 안의 지역변수를 읽고 수정할 수 있으나 람다에서는 사실상 final인 변수만 읽을 수 있고, 지역변수 수정은 불가능

- 코드 블록에서는 return, break, continue를 사용 가능하며 메서드 선언에 명시된 검사 예외 던지기 가능 (람다는 불가능)


<br>

__❤️‍🩹 스트림에 안성 맞춤인 예시__

- 원소들의 시퀀스를 일관되게 변환

- 원소들의 시퀀스를 필터링
- 원소들의 시퀀스를 하나의 연산을 사용해 결합 (더하기, 연결하기, 최솟값 구하기 등)
- 원소들의 시퀀스를 컬렉션에 모으기 (공통된 속성을 기준으로)
- 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾기


<br>

__❤️‍🩹 스트림으로 처리하기 어려운 일__

한 데이터가 파이프라인의 여러 단계를 통과할 때 데이터의 각 단계에서 값들에 동시 접근하기 어려운 경우,

한 값을 다른 닶에 매핑하면 원래의 값을 잃게 되는 구조이기 때문에 스트림으로 처리하기 어렵다.

원래 값과 새로운 값의 쌍을 저장하는 객체를 사용해 우회할 수 있지만, 좋은 방법은 아니다.
