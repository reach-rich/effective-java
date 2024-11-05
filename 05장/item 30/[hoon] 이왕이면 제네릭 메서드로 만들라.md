## 1. 들어가기

지난 Item29에서는 제네릭을 클래스에 적용하는 방법을 알아보았습니다.

이번에는 제네릭을 메서드에 적용하는 방법에 대해 알아보겠습니다.

## 2. 제네릭 메서드

제네릭 메서드의 작성법은 제네릭 타입 작성법과 비슷합니다.

Raw 타입을 사용한 메서드를 제네릭 메서드로 변경하는 예시를 통해 작성법을 알아봅시다.

```java
/* Raw 타입을 사용한 메서드 */
  public static Set union(Set s1, Set s2) {
    Set result = new HashSet(s1);
    result.addAll(s2);
    return result;
  }
```

해당 코드는 컴파일은 되지만 다음의 경고가 발생합니다.

```
  Main.java:9: warning: [unchecked] unchecked call to HashSet(Collection<? extends E>) as a member of the raw type HashSet
        Set result = new HashSet(s1);
                     ^
  where E is a type-variable:
    E extends Object declared in class HashSet

  Main.java:10: warning: [unchecked] unchecked call to addAll(Collection<? extends E>) as a member of the raw type Set
        result.addAll(s2);
                     ^
  where E is a type-variable:
    E extends Object declared in interface Set
```

경고를 없애기 위해서는 타입을 안전하게 만들어야 합니다.

타입을 안전하게 만들기 위해서는 원소 타입을 타입 매개변수로 명시하고

메서드 내부에서도 해당 타입 매개변수만 사용하게 수정하면 됩니다.

다음은 위의 Raw 타입 메서드를 타입 안전하게 만든 제네릭 메서드 예시입니다.

```java
  public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
    Set<E> result = new HashSet<>();
    result.addAll(s2);
    return result;
  }
```

## 3. 제네릭 싱글턴 팩터리

때때로 불변 객체를 여러 타입으로 만들어야 하는 경우가 있습니다.

이런 경우, 요청한 타입 매개변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩터리가 필요한데

이러한 패턴을 **제네릭 싱글턴 팩터리**라고 합니다.

```java
  private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

  @SuppressWarnings("unchecked")
  public static <T> UnaryOperator<T> identityFunction() {
    return (UnaryOperator<T>) IDENTITY_FN;
  }
```

```java
  UnaryOperator<String> sameString = identityFunction();
  UnaryOperator<Number> sameNumber = identityFunction();
```

## 4. 재귀적 타입 한정

자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정할 수도 있습니다.

바로 재귀적 타입 한정(Recursive Type Bound)이라는 개념입니다.

재귀적 타입 한정은 주로 Comparable 인터페이스와 함께 쓰입니다.

```java
  public interface Comparable<T> {
    int compareTo(T o);
  }
```

Comparable을 구현한 메서드들은 주로 정렬, 검색 또는 최대값/최소값을 구하는데 사용됩니다.

이 기능을 수행하기 위해서는 컬렉션의 모든 원소가 상호 비교될 수 있어야 하는데

이 제약을 코드로 표현하면 다음과 같습니다.

```java
  public static <E extends Comparable<E>> E max(Collection<E> c);
```

여기서 재귀적 타입 한정은 바로 `<E extends Comparable<E>>` 부분인데,

"모든 타입 E는 자신과 비교할 수 있다" 라고 해석할 수 있습니다.

## 5. 정리

이번 포스트는 제네릭을 메서드에 적용하는 법을 알아보았습니다.

Object 타입 메서드는 클라이언트에서 사용할 때마다 형변환을 해줘야 하지만,

제네릭 메서드는 형변환 없이 사용할 수 있으므로 더 안전하고 사용하기 쉽습니다.

제네릭 타입과 마찬가지로 이왕이면 **제네릭 메서드**로 설계하고,

기존 메서드 중 형변환을 해줘야 하는 메서드는 제네릭 메서드로 변경합시다.