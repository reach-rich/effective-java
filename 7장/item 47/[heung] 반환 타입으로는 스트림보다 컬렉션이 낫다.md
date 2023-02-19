Effective Java의  47번째 아이템 "반환 타입으로는 스트림보다 컬렉션이 낫다"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 1. 원소 시퀀스를 반환하는 메서드 타입

원소 시퀀스, 즉 일련의 원소를 반환하는 메서드는 수없이 많다. 자바 7까지는 이런 메서드의 반환 타입으로 Collection, Set, List 같은 컬렉션 인터페이스, 혹은 Iterable이나 배열을 썼다. 이 중 가장 적합한 타입을 선택하는 방법은 컬렉션 인터페이스를 기본으로 하되, for-each 문에서만 쓰이거나 반환된 원소 시퀀스가 일부 Collection 메서드를 구현할 수 없을 때는 Iterable 인터페이스를 사용했다. 그런데 **자바 8이 스트림이라는 개념을 들고 오면서 이 선택이 아주 복잡한 일이 되어버렸다.**

<br>

### 1) API가 스트림만 반환하는 경우

스트림은 반복을 지원하지 않는다. 따라서 스트림과 반복을 알맞게 조합해야 좋은 코드가 나온다. 

API가 스트림만 반환하도록 짜놓으면 반환된 스트림을 for-each로 반복하길 원하는 사용자는 불만을 토로할 것이다. 아래는 스트림을 반복하기 위한 우회 방법이다.

```java
for (ProcessHandle ph : (Iterable<ProcessHandle>) ProcessHandle.allProcesses()::iterator) {
  // 프로세스를 처리한다.
}
```

작동은 하지만 실전에 쓰기에는 너무 난잡하고 직관성이 떨어진다. 다행히 어댑터 메서드를 사용하면 상황이 나아진다. 어댑터를 사용하면 어떤 스트림도 for-each 문으로 반복할 수 있다. 

```java
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
  return stream::iterator;
}
```

```java
for (ProcessHandle ph : iterableOf(ProcessHandle.allProcesses())) {
  // 프로세스를 처리한다.
}
```

<br>

### 2) API가 Iterable만 반환하는 경우

API가 Iterable만 반환하면 이를 스트림 파이프라인에서 처리하려는 프로그래머가 성을 낼 것이다. 이 경우에도 마찬가지로 어댑터를 사용할 수 있다. 

```java
public static <E> Stream<E> streamOf(Iterable<E> iterable) {
  return StreamSupport.stream(iterable.spliterator(), false);
}
```

<br>

### 3) API가 Collection을 반환하는 경우

Collection 인터페이스는 Iterable의 하위 타입이고 stream 메서드도 제공하니 반복과 스트림을 동시에 지원한다. 따라서 **원소 시퀀스를 반환하는 공개 API의 반환 타입에는 Collection이나 그 하위 타입을 쓰는 게 일반적으로 최선이다.**

반환하는 시퀀스의 크기가 메모리에 올려도 안전할 만큼 작다면 ArrayList나 HashSet 같은 표준 컬렉션 구현체를 반환하는 게 최선일 수 있다. 하지만 **단지 컬렉션을 반환한다는 이유로 덩치 큰 시퀀스를 메모리에 올려서는 안 된다.** 

<br>

## 2. 전용 컬렉션

반환할 시퀀스가 크지만 표현을 간결하게 할 수 있다면 전용 컬렉션을 구현하는 방안을 검토해보자. 예를 들어 주어진 집합의 멱집합(한 집합의 모든 부분집합을 원소로 하는 집합)을 반환하는 상황이다. 

원소의 개수가 n개면 멱집합의 원소 개수는 2^n개가 된다. 그러니 멱집합을 표준 컬렉션 구현체에 저장하려는 생각은 위험하다. 하지만 AbstractList를 이용하면 훌륭한 전용 컬렉션을 손쉽게 구현할 수 있다. 

비결은 멱집합을 구성하는 각 원소의 인덱스를 비트 벡터로 사용하는 것이다. 인덱스의 n번째 비트 값은 멱집합의 해당 원소가 원래 집합의 n번째 원소를 포함하는지 여부를 알려준다.

```java
public class PowerSet {
  public static final <E> Collection<Set<E>> of(Set<E> s) {
    List<E> src = new ArrayList<>(s);
    if (src.size() > 30) {
      throw new IllegalArgumentException("집합에 원소가 너무 많습니다.(최대 30개): " + s);
    }
    
    return new AbstractList<Set<E>>() {
      @Override public int size() {
        return 1 << src.size();
      }
      
      @Override public boolean contains(Object o) {
        return o instanceof Set && src.containsAll((Set)o);
      }
      
      @Override public Set<E> get(int index) {
        Set<E> result = new HashSet<>();
        for (int i = 0; index != 0; i++, index >>= 1) {
          if ((index & 1) == 1) {
            result.add(src.get(i));
          }
        }
        return result;
      }
    };
  }
}
```

AbstractList를 사용하여 Collection 구현체를 작성할 때는 Iterable용 메서드 외에 2개만 더 구현하면 된다. 바로 contains와 size다. 

containse와 size를 구현하는 게 불가능할 때는 컬렉션보다는 Stream이나 Iterable을 반환하는 편이 낫다. 원한다면 별도의 메서드를 두어 두 방식을 모두 제공해도 된다. 

<br>

> * Collection의 단점
>
> Collection의 size 메서드가 int 값을 반환하므로 PowerSet.of가 반환하는 시퀀스의 최대 길이는 Integer.MAX_VALUE 혹은 2^31-1로 제한된다. Collection 명세에 따르면 컬렉션이 더 크거나 심지어 무한대일 때 size가 2^31-1을 반환해도 되지만 완전히 만족스러운 해법은 아니다. 

<br>

## 3. 핵심 정리

* 원소 시퀀스를 반환하는 메서드를 작성할 때는, 이를 스트림으로 처리하기를 원하는 사용자와 반복으로 처리하길 원하는 사용자가 모두 있을 수 있음을 떠올리고, 양쪽을 다 만족시키려 노력하자.
* 컬렉션을 반환할 수 있다면 그렇게 하라.
* 반환 전부터 이미 원소들을 컬렉션에 담아 관리하고 있거나 컬렉션을 하나 더 만들어도 될 정도로 원소 개수가 적다면 ArrayList 같은 표준 컬렉션에 담아 반환하라. 그렇지 않으면 전용 컬렉션을 구현할지 고민하라.
* 컬렉션을 반환하는 게 불가능하면 스트림과 Iterable 중 더 자연스러운 것을 반환하라.
