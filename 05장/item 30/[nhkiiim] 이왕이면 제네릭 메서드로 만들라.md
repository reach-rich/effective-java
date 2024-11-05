# 이왕이면 제네릭 메서드로 만들라
- 클래스처럼 메서드도 제네릭으로 만들 수 있음
- 매개변수화 타입을 받는 정적 유틸리티 메서드는 보통 제네릭!
- Collections 의 알고리즘 메서드도 모두 제네릭

### 1. 제네릭 메서드

```java
public static Set union(Set s1, Set s2) {
    Set result = new HashSet(s1);
    result.addAll(s2);
    return result;
}
```

- 로 타입 사용으로 컴파일은 되지만 경고 발생

```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
    Set<E> result = new HashSet(s1);
    result.addAll(s2);
    return result;
}

public static void main(String[] args) {
    Set<String> guys = Set.of("톰", "딕", "해리");
    Set<String> stooges = Set.of("래리", "모에", "컬리");
    Set<String> aflCio = union(guys, stooges);
    System.out.println(aflCio);
}
```

- 제네릭 메서드! 경고 없이 컴파일, 타입 안전, 쓰기도 쉬움
- 하지만 union 메서드는 입력 2개 반환 1개의 타입이 모두 같아야함
- 이를 와일드카드 타입을 사용해 더 유연하게 사용 가능

#
### 2. 제네릭 싱글턴 패턴

- Collections.reverseOrder 같은 함수 객체나 Collections.emptySet 같은 컬렉션용으로 사용
- 항등함수를 담은 클래스를 만들고 싶다 가정
- Function.identity를 사용하면 되지만 직접 작성하며 공부해 ^^
- 항등함수 객체는 상태가 없어서 요청할 때 마다 새로 생성하는 건 낭비임

```java
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressWarnings("unchecked")
public static <T> UnaryOperator<T> identityFunction() {
    return (UnaryOperator<T>) IDENTITY_FN;
}
```
- 이해가 잘 안됨..???


#
### 3. 재귀적 타입 한정

```java
public interface Comparable<T> {
    int compareTo(T o);
}
```

- 여기서 타입매개변수 T는 Comparable\<T>를 구현한 타입이 비교할 수 있는 원소의 타입을 정의
- 실제로 거의 모든 타입은 자신과 같은 타입의 원소와만 비교 가능
- String은 Comparable\<String>을 구현, Integer는 Comparable\<Integer>를 구현
- Comparable은 원소의 컬렉션을 입력받는 메서드로 정렬 혹은 검색 -> 컬렉션에 담긴 모든 원소가 상호 비교될 수 있어야 한다!


```java
public static <E extends Comparable<E>> E max(Collection<E> c);
```
- 재귀적 타입 한정을 이용해 상호 비교 가능

```java
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if (c.isEmpty())
        throw new IllegalArgumentException("컬렉션이 비어 있습니다.");
 
    E result = null;
    for (E e : c)
        if (result == null || e.compareTo(result) > 0)
            result = Objects.requreNonNull(e);
    
    return result;
}
```
