Effective Java의 서른 번째 아이템 "이왕이면 제네릭 메서드로 만들라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 1. 제네릭 메서드 작성법

클래스와 마찬가지로, 메서드도 제네릭으로 만들 수 있다. 제네릭 메서드 작성법은 제네릭 타입 작성법과 비슷하다. 다음은 두 집합의 합집합을 반환하는, 문제가 있는 메서드다.

```java
public static Set union(Set s1, Set s2) {
  Set result = new HashSet(s1);
  result.addAll(s2);
  return result;
}
```

컴파일은 되지만 경고가 두 개 발생한다.

`warning: Set result = new HashSet(s1);`
`warning: result.addAll(s2);`

경고를 없애려면 이 메서드를 타입 안전하게 만들어야 한다. 메서드 선언에서의 세 집합(입력 2개, 반환 1개)의 원소 타입을 타입 매개변수로 명시하고, 메서드 안에서도 이 타입 매개변수만 사용하게 수정하면 된다.

<br>

(타입 매개변수들을 선언하는) 타입 매개변수 목록은 메서드의 제한자와 반환 타입 사이에 온다. 다음 코드에서 타입 매개변수 목록은 \<E\>이고 반환 타입은 Set\<E\>이다. 타입 매개변수의 명명 규칙은 제네릭 메서드나 제네릭 타입이나 똑같다.

 ```java
 public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
   Set<E> result = new HashSet<>(s1);
   result.addAll(s2);
   return result;
 }
 ```

단순한 제네릭 메서드라면 이 정도면 충분하다. 이 메서드는 경고 없이 컴파일되며, 타입 안전하고, 쓰기도 쉽다.

위의 메서드는 집합 3개(입력 2개, 반환 1개)의 타입이 모두 같아야 한다. 이를 한정적 와일드카드 타입을 사용하여 더 유연하게 개선할 수 있다.

<br>

## 2. 제네릭 싱글턴 팩터리 패턴

때때로 불변 객체를 여러 타입으로 활용할 수 있게 만들어야 할 때가 있다. 제네릭은 런타임에 타입 정보가 소거되므로 하나의 객체를 어떤 타입으로든 매개변수화할 수 있다. 하지만 이렇게 하려면 요청한 타입 매개변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩터리로 만들어야 한다. 이 패턴을 제네릭 싱글턴 팩터리라 한다.

<br>

항등함수를 담은 클래스를 만들고 싶다고 해보자.

항등함수 객체는 상태가 없으니 요청할 때마다 새로 생성하는 것은 낭비다. 자바의 제네릭이 실체화된다면 항등함수를 타입별로 하나씩 만들어야 했겠지만, 소거 방식을 사용한 덕에 제네릭 싱글턴 하나면 충분하다. 

```java
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressWarnings("unchecked")
public static <T> UnaryOperator<T> identityFunction() {
  return (UnaryOperator<T>) IDENTITY_FN;
}
```

IDENTITY_FN을 UnaryOperator\<T\>로 형변환하면 비검사 형변환 경고가 발생한다. T가 어떤 타입이든 UnaryOperator\<Object\>는 UnaryOperator\<T\>가 아니기 때문이다. 

하지만 항등함수란 입력 값을 수정 없이 그대로 만환하는 특별한 함수이므로, T가 어떤 타입이든 UnaryOperator\<T\>를 사용하도 타입 안전하다. 우리는 이 사실을 알고 있으니 이 메서드가 내보내는 비검사 형변환 경고는 숨겨도 안심할 수 있다.

<br>

## 3. 재귀적 타입 한정

상대적으로 드물긴 하지만, 자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정할 수 있다. 바로 재귀적 타입 한정이라는 개념이다. 재귀적 타입 한정은 주로 타입의 자연적 순서를 정하는 Comparable 인터페이스와 함께 쓰인다. 

```java
public interface Comparable<T> {
  int compareTo(T o);
}
```

여기서 타입 매개변수 T는 Comparable\<T\>를 구현한 타입이 비교할 수 있는 원소의 타입을 정의한다. 

<br>

Comparable을 구현한 원소의 컬렉션을 입력받는 메서드들은 주로 그 원소들을 정렬 혹은 검색하거나, 최솟값이나 최댓값을 구하는 식으로 사용된다. 이 기능을 수행하려면 컬렉션에 담긴 모든 원소가 상호 비교될 수 있어야 한다.

다음은 이 제약을 코드로 표현한 모습이다.

```java
public static <E extends Comparable<E>> E max(Collection<E> c);
```

타입 한정인 \<E extends Comparable\<E\>\>는 "모든 타입 E는 자신과 비교할 수 있다"라고 읽을 수 있다. 상호 비교 가능하다는 뜻을 아주 정확하게 표현했다.

다음은 방금 선언한 메서드의 구현이다. 컬렉션에 담긴 원소의 자연적 순서를 기준으로 최댓값을 계산하며, 컴파일 오류나 경고는 발생하지 않는다.

```java
public static <E extends Comparable<E>> E max(Collection<E> c) {
  if (c.isEmpty())
    throw new IllegalArgumentException("컬렉션이 비어 있습니다.");
  
  E result = null;
  for (E e : c) {
    if (result == null || e.compareTo(result) > 0)
      result = Objects.requireNonNull(e);
  }
  
  return result;
}
```

재귀적 타입 한정은 훨씬 복잡해질 가능성이 있긴 하지만, 다행히 그런 일은 잘 일어나지 않는다. 이번 아이템에서 설명한 관용구, 여기에 와일드카드를 사용한 변형, 그리고 시뮬레이트한 셀프 타입 관용구를 이해하고 나면 실전에서 마주치는 대부분의 재귀적 타입 한정을 무리 없이 다룰 수 있을 것이다.

<br>

## 4. 핵심 정리

- 클라이언트에서 입력 매개변수와 반환값을 명시적으로 형변환해야 하는 메서드보다 제네릭 메서드가 더 안전하며 사용하기도 쉽다.
- 메서드도 형변환 없이 사용할 수 있는 편이 좋으며, 많은 경우 그렇게 하려면 제네릭 메서드가 되어야 한다.

<br>

## 5. Related Posts

- 제네릭 타입 (Item 29)
- 변수의 명명 규칙 (Item 68)
- 한정적 와일드카드 타입 (Item 31)
- [타입 정보 소거 (Item 28)](https://heung27.github.io/posts/item-28-%EB%B0%B0%EC%97%B4%EB%B3%B4%EB%8B%A4%EB%8A%94-%EB%A6%AC%EC%8A%A4%ED%8A%B8%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC/)
- [Comparable (Item 14)](https://heung27.github.io/posts/item-14-comparable%EC%9D%84-%EA%B5%AC%ED%98%84%ED%95%A0%EC%A7%80-%EA%B3%A0%EB%A0%A4%ED%95%98%EB%9D%BC/)
- [셀프 타입 관용구 (Item 2)](https://heung27.github.io/posts/effective-java-item-2-%EC%83%9D%EC%84%B1%EC%9E%90%EC%97%90-%EB%A7%A4%EA%B0%9C%EB%B3%80%EC%88%98%EA%B0%80-%EB%A7%8E%EB%8B%A4%EB%A9%B4-%EB%B9%8C%EB%8D%94%EB%A5%BC-%EA%B3%A0%EB%A0%A4%ED%95%98%EB%9D%BC/)
