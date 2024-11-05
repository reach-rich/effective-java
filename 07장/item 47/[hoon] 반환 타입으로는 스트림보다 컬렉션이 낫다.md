## 1. 들어가기

일련의 원소를 반환하는 메서드는 수없이 많습니다.

Java 8 이전에는 이러한 메서드 반환 타입으로 컬렉션 인터페이스나 Iterable 또는 배열을 썼는데

이 중 가장 적합한 타입을 선택하기란 그다지 어렵지 않았습니다.

하지만 Java 8 이후부터 Stream이라는 개념이 도입되면서 복잡한 일이 되었습니다.

## 2. Stream의 문제

만약, Stream을 반환하는 메서드를 사용한다면 반환된 Stream을 for-each 구문으로 반복할 수 없습니다.

그 이유는 Iterable을 확장(extend)하지 않았기 때문입니다.

```java
/* 컴파일 오류 */
   for(ProcessHandle ph : ProcessHandle.allProcesses()) {
      ...
   }
```

그럼 Stream의 iterator 메서드에 메서드 참조를 건네면 되지 않을까 해서 다음과 같이 작성할 수 있습니다.

하지만 다음의 코드도 마찬가지로 컴파일 오류가 일어납니다.

```java
/* 컴파일 오류 */
   for(ProcessHandle ph : ProcessHandle.allProcesses()::iterator) {
      ...
   }
```

이러한 오류를 바로잡기 위해 다음과 같이 형변환을 할 수도 있으나

실전에서 쓰기엔 너무 난잡하고 직관성이 떨어져 좋은 방법은 아닙니다.

```java
   for(ProcessHandle ph : (Iterable<ProcessHandle>) ProcessHandle.allProcesses()::iterator) {
      ...
   }
```

가장 좋은 방법은 **어댑터**를 사용하는 방법입니다.

어댑터는 한 인터페이스를 다른 인터페이스로 변환하는 디자인 패턴으로

인터페이스 호환성 문제로 사용할 수 없는 클래스를 연결해서 사용할 수 있습니다.

어댑터를 이용하면 다음과 같이 사용할 수 있습니다.

```java
   public static <E> Iterable<E> iterableOf(Stream<E> stream) {
      return stream::iterator;
   }

   for(ProcessHandle ph : iterableOf(ProcessHandle.allProcesses())) {
      ...
   }
```

하지만, 반대로 API가 Iterable만 반환하면 이를 Stream Pipeline에서 처리할 수 없습니다.

그래서 이를 위한 어댑터를 다음과 같이 제공할 수 있습니다.

```java
   public static <E> Stream<E> streamOf(Iterable<E> iterable) {
      return StreamSupport.stream(iterable.spliterator(), false);
   }
```

## 3. Collection

위의 코드에서는 Stream과 반복문을 사용하는 사람 모두를 배려하기 위해 두 어댑터를 제공했습니다.

이처럼 공개 API를 작성할 때는 한 방식만 사용하지는 않기 때문에 모든 사용자를 배려해야 합니다.

Collection 인터페이스는 Iterable의 하위 타입이고 stream 메서드도 제공하므로 둘을 동시에 지원합니다.

따라서, 원소 시퀀스를 반환하는 공개 API의 반환 타입은 컬렉션이나 그 하위 타입을 쓰는게 좋습니다.

## 4. 전용 Collection을 사용하는 경우

하지만 단지 컬렉션을 반환한다는 이유로 덩치 큰 원소를 그대로 표준 컬렉션 구현체로 반환해서는 안됩니다.

만약 그럴 경우에는 전용 컬렉션을 구현하는 방법을 검토합시다.

예를 들어 주어진 집합의 멱집합을 반환하는 상황을 있다고 가정해봅시다.

`{a, b, c}`의 멱집합은 `{ {}, {a}, {b}, {c}, {a, b}, {a, c}, {b, c}, {a, b, c} }` 입니다.

원소의 개수가 n개면 멱집합의 원소 개수는 2ⁿ개가 됩니다.

그렇기 때문에 멱집합을 표준 컬렉션 구현체에 저장하는 것은 매우 위험합니다.

하지만 전용 컬렉션을 구현하면 손쉽게 구현할 수 있습니다.

```java
   public class PowerSet {
      public static final <E> Collection<Set<E>> of(Set<E> s) {
         List<E> src = new ArrayList<>(s);
         if(src.size() > 30) {
            throw new IllegalArgumentException("집합에 원소가 너무 많습니다(최대 30개).: " + s);
         }

         return new AbstractList<Set<E>>() {
            @Override
            public int size() {
               return 1 << src.size();
            }

            @Override
            public boolean contains(Object o) {
               return o instanceof Set && src.containsAll((Set) o);
            }

            @Override
            public Set<E> get(int index) {
               Set<E> result = new HashSet<>();
               for (int i = 0; index != 0; i++, index >>=1) {
                  if((index & 1) == 1) {
                     result.add(src.get(i));
                  }
               }
               return result;
            }
         };
      }
   }
```

전용 컬렉션을 구현할 때는 Iterable용 메서드 외에 contains와 size 메서드만 더 구현하면 됩니다.

만약, contains와 size 메서드를 구현하는게 불가능할 때는 stream이나 iterable을 반환하는게 낫습니다.

## 5. Stream을 사용하는 경우

때로는 단순히 구현하기 쉬운 쪽을 선택하기도 합니다.

예를 들어 입력 리스트의 부분리스트를 모두 반환하는 메서드를 작성한다고 가정해봅시다.

컬렉션을 사용하면 입력 리스트 크기의 거듭제곱만큼 메모리를 차지하기 때문에 좋은 방법이 아닙니다.

하지만 이를 Stream으로 구현했을 때는 지연 연산을 하므로 컬렉션보다 많이 차지하지 않을 뿐더러

그 리스트의 prefix와 suffix에 빈 리스트 하나만 추가하면 되므로 매우 직관적으로 구현할 수 있습니다.

```java
   public class SubList {
      public static <E> Stream<List<E>> of(List<E> list) {
         return Stream.concat(Stream.of(Collections.emptyList()), prefixes(list).flatMap(SubList::suffixes));
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

이를 for 반복문으로 중첩해 만들면 다음과 같습니다.

```java
   for (int start = 0; start < src.size(); start++) {
      for (int end = start + 1; end <= src.size(); end++) {
         System.out.println(src.subList(start, end));
      }
   }
```

이 반복문을 그대로 Stream으로 변환할 수 있지만, 맨 처음 구현보다 간결해지나 가독성이 좋지 않습니다.

```java
   public static <E> Stream<List<E>> of(List<E> list) {
      return IntStream.range(0, list.size())
         .mapToObj(start -> 
                     IntStream.rangeClosed(start + 1, list.size())
                              .mapToObj(end -> list.subList(start, end)))
         .flatMap(x -> x);
   }
```

## 6. 정리

일련의 원소를 반환하는 메서드를 작성할 때는 반복과 Stream을 사용하는 사용자가 있기 때문에

양쪽을 모두 만족시키기 위해 노력합시다.

이왕이면 반복과 Stream 모두 호환되는 표준 Collection을 반환하는 것이 좋습니다.

하지만, 멱집합 예처럼 전용 Collection을 구현할지도 고민해봐야 합니다.

Collection을 반환하는 것이 불가능하다면 Stream과 Iterable 중 더 자연스러운 것을 반환합시다.