### 🤔스트림의 반복 지원

- 스트림은 반복을 지원하지 않음
- `Stream`인터페이스는`Iterable` 인터페이스가 정의한 추상 메서드를 전부 포함할 뿐 아니라, `Iterable` 인터페이스가 정의한 방식대로 동작
- `for-each`로 스트림을 반복할 수 없는 이유는 `Stream`이 `Iterable`을 확장하지 않음

<br>

---

<br>

### 💡 해결방법

1️⃣**Stream의 iterator 메서드 참조를 건네는 방법**

**✏ #01 예제소스 | 자바 타입 추론의 한계로 컴파일되지 않는다**

```java
for (ProcessHandle ph : ProcessHandle.allProcesses()::iterator) {
    // 프로세스를 처리한다
}
```

>컴파일 오류 발생!

**✏ #02 예제소스 | 스트림을 반복하기 위한 '끔찍한' 우회 방법**

```java
for (ProcessHandle ph : (Iterable<ProcessHandle>)ProcessHandle.allProcesses()::iterator) {
    // 프로세스를 처리한다
}
```

>작동은 하지만 너무 난잡하고 직관성이 떨어짐

**✏ #03 예제소스 | Stream<E>를 Iterable<E>로 중개해주는 어댑터**

```java
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
    return stream::iterator;
}
```

>자바의 타입 추론이 문맥을 잘 파악하여 어댑터 메서드 안에서 따로 형변환 필요 없음
>
>어댑터를 사용하면 어떤 스트림도 `for-each` 문으로 반복 가능

**✏ #04 예제소스 | Iterable<E>를 Stream<E>를 중개해주는 어댑터**

```java
public static <E> Stream<E> streamOf(Iterable<E> iterable) {
	return StreamSupport.stream(iterable.spliterator(), false);
}
```

>이 메서드가 오직 스트림 파이프라인에서만 쓰일 걸 안다면 편하게 스트림을 반환
>
>반대로 반환된 객체들이 반복문에서만 쓰일 걸 안다면` Iterable`을 반환
>
>하지만 API 작성 시에는 모두 고려해줘야 함

<br>

**2️⃣ Collection 인터페이스는 Iterable의 하위 타입이고 stream 메서드도 제공하니 반복과 스트림 동시에 지원**

>원소 시퀀스를 반환하는 공개 API의 반환 타입에는 `Collection`이나 그 하위 타입을 쓰는 게 일반적으로 최선

<br>

**3️⃣ Arrays 역시 Arrays.asList와 Stream.of 메서드로 반복과 스트림 지원**

<br>

**✔ 하지만 단지 컬렉션을 반환한다는 이유로 덩치 큰 시퀀스를 메모리에 올려서는 안됨! **

<br>

**4️⃣ AbstractList를 이용한 전용 컬렉션**

>반환할 시퀀스가 크지만 표현을 간결하게 할 수 있다면 전용 컬렉션을 구현
>
>ex.멱집합을 반환하는 상황
>
>멱집합을 구성하는 각 원소의 인덱스를 비트 벡터로 사용

**✏ #05 예제소스 | 입력 집합의 멱집함을 전용 컬렉션에 담아 반환**

```java
public class PowerSet{
    public static final <E> Collection<Set<E>> of(Set<E> s){
        List<E> src = new ArrayList<>(s);
        if (src.size() > 30) {
            throw new IllegalArgumentException(
                  "집합에 원소가 너무 많습니다.(최대 30개).:" + s);

        return new AbstractList<Set<E>>(){
            @Override
            public int size(){
                // 멱집합의 크기는 2를 원래 집합의 원소 수만큼 거듭제곱한 것과 같다
                return 1 << src.size();
            }
            @Override
            public boolean contains(Object o){
                return o instanceof Set && src.containsAll((Set)o);
            }
            @Override
            public Set<E> get(int index){
                Set<E> result = new HashSet<>();
                for(int i = 0; index != 0; i++, index >>= 1){
                    if ((index&1)==1)
                        result.add(src.get(i));
                }
                return result; 
            }
        }
    }
}
```

>입력 집합의 원소 수가 30을 넘으면 `PowerSet.of`가 예외를 던짐
>
>`Stream`이나 `Iterable`이 아닌 `Collection`을 반환 타입으로 쓸 때의 단점

<br>

**5️⃣ AbstractCollection을 활용한 Collection구현체**

>`AbstractCollection`을 활용해서 `Collection` 구현체를 작성할 때 `Itreable`용 메서드 외에 `contains`와 `size`를 추가 구현
>
>`contains`와 `size`를 구현하는 게 불가능할 경우 컬렉션보다는 스트림이나 `Iterable`을 반환하는 편이 나음
>
>원한다면 별도의 메서드를 두어 두 방식을 모두 제공해도 됨

<br>

**8️⃣ 전용 컬렉션보다 간단하게 구현하는 방법**

**✏ #06 예제소스 | 입력 리스트의 모든 부분리스트를 스트림으로 반환한다**

```java
public class SubList {
    public static <E> Stream<List<E>> of(List<E> list) {
        return Stream.concat(Stream.of(Collections.emptyList()), 
                             prefixes(list).flatMap(SubList::suffixes));
    }

    public static <E> Stream<List<E>> prefixes(List<E> list) {
        return IntStream.rangeClosed(1, list.size())
                        .mapToObj(end -> list.subList(0, end));
    }

    public static <E> Stream<List<E>> suffixes(List<E> list) {
        return IntStream.rangeClosed(0, list.size())
                        .mapToObj(start -> list.subList(start, list.size()));
    }
}
```

<br>

**✏ #07 예제소스 | 같은 역할의 for 반복문을 중첩해 만든 것**

```java
for (int start = 0; start < src.size(); start++) {
    for (int end = start + 1; end <= src.size(); end++) {
        System.out.println(src.subList(start, end));
    }
}
```

<br>

**✏ #08 예제소스 | for 반복문을 중첩 스트림으로 변환**

```java
public static <E> Stream<List<E>> of(List<E> list) {
    return IntStream.range(0, list.size())
        .mapToObj(start -> 
                  IntStream.rangeClosed(start + 1, list.size())
                           .mapToObj(end -> list.subList(start, end)))
        .flatMap(x -> x);
}
```

<br>

----

<br>

### 📌 핵심정리

**원소 시퀀스를 반환하는 메서드를 작성할 때는, 이를 스트림으로 처리하기를 원하는 사용자와 반복으로 처리하길 원하는 사용자가 모두 있을 수 있음을 떠올리고, 양쪽을 다 만족시키려 노력하자**

**컬렉션을 반환할 수 있다면 그렇게 하라**

**반환 전부터 이미 원소들을 컬렉션에 담아 관리하고 있거나 컬렉션을 하나 더 만들어도 될 정도로 원소 개수가 적다면 ArrayList 같은 표준 컬렉션에 담아 반환하라**

**그렇지 않으면 앞서의 멱집합 예처럼 전용 컬렉션을 구현할지 고민하라**

**컬렉션을 반환하는 게 불가능하면 스트림과 Iterable 중 더 자연스러운 것을 반환하라**

**만약 나중에 Stream 인터페이스가 Iterable을 지원하도록 자바가 수정된다면, 그때는 안심하고 스틀미을 반환하면 될 것이다(스트림 처리와 반복 모두에 사용할 수 있으니)**

<br>
