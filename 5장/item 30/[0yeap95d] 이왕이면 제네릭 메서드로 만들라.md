### 🔎 제네릭 메서드

- 메서드를 제네릭으로 만들 수 있다
- 매개변수화 타입을 받는 정적 유틸리티 메서드는 보통 제네릭
- 예시로 `Collections`의 알고리즘 메서드는 모두 제네릭

<br>

**✏ #01 예제소스 | 로 타입 사용 - 수용 불가!**

```java
public static Set union(Set s1, Set s2) {
    Set result = new HashSet(s1);
    result.addAll(s2);
    return result;
}
```

>**❗ 2가지 경고 발생**
>
>```tex
>Union.java:5: warning: [unchecked] unchecked call to
>HashSet(Collection<? extends E>) as a member of raw type HashSet
>	Set result = new HashSet(s1);
>				 ^
>```
>
>```tex
>Union.java:5: warning: [unchecked] unchecked call to
>addAll(Collection<? extends E>) as a member of raw type Set
>	result.addAll(s2);
>		      ^
>```
>
>- 경고를 없애려면 타입 안전하게 만들어야 한다
>- 메서드 선언에서의 세 집합(입력 2개, 반환 1개) 원소 타입을 타입 매개변수로 명시하고, 메서드 안에서도 이 타입 매개변수만 사용하게 수정
>- 타입 매개변수 목록은 메서드의 제한자와 반환 타입 사이에 온다

<br>

**✏ #02 예제소스 | 제네릭 메서드**

```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
    Set<E> result = new HashSet(s1);
    result.addAll(s2);
    return result;
}
```

>- 단순한 제네릭 메서드라면 이 정도 충분
>- 경고 없이 컴파일 되며, 타입 안전하고, 쓰기도 쉬움
>- 한정적 와일드카드 타입을 사용하여 더 유연하게 개선 가능

<br>

---

<br>

### 💡 제네릭 싱글턴 팩터리

- 불변 객체를 여러 타입으로 활용할 수 있도록 만들어야 하는 경우 사용
- 제네릭은 런타임에 타입 정보가 소거되므로 하나의 객체를 어떤 타입으로든 매개변수화 가능
- 요청한 타입 매개변수에 맞게 매번 그 객체를 바꿔주는 정적 팩토리 필요 (= 제네릭 싱글턴 팩터리)
- `Collections.reverseOrder` 같은 함수객체나 `Collections.emptySet` 같은 컬렉션용으로 사용

<br>

---

<br>

### 📒 항등함수를 담은 클래스 (Function.identity)

- 항등함수 객체는 상태가 없으니 요청할 때 마다 새로 생성하는 것은 낭비
- 소거 방식을 사용한 덕에 제네릭 싱글턴으로 충분

**✏ #03 예제소스 | 제네릭 싱글턴 팩터리 패턴**

```java
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressWarnings("unchecked")
public static <T> UnaryOperator<T> identityFunction() {
    return (UnaryOperator<T>) IDENTITY_FN;
}
```

>`IDENTITY_FN`을 `UnaryOperator<T>`로 형변환하면 비검사 형변환 경고가 발생
>
>`T`가 어떤 타입이든 `UnaryOperator<Object>`는 `UnaryOperator<T>`가 아니기 때문
>
>하지만 항등함수란 입력 값을 수정 없이 그대로 반환하는 특별한 함수이므로 `T`가 어떤 타입이든 `UnaryOperator<T>`를 사용해도 타입 안전함
>
>따라서 `@SuppressWarnings`애너테이션을 추가하면 오류나 경고 없이 컴파일 가능

<br>

---

<br>

### 📙 재귀적 타입 한정(recursive type bound)

- 상대적으로 드물긴 하지만, 자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정
- 주로 타입의 자연적 순서를 정하는 Comparable 인터페이스와 함께 쓰임

**✏ #04 예제소스**

```java
public interface Comparable<T> {
	int compareTo(T o);
}
```

>타입 매개변수 `T`는 `Comparable<T>`를 구현한 타입이 비교할 수 있는 원소의 타입을 정의
>
>실제로 거의 모든 타입은 자신과 같은 타입의 원소와만 비교 가능
>
>`String`은 `Comparable<String>`을 구현하고, `Integer`는 `Comparable<Integer>`를 구현하는 방식
>
>`Comparable`을 구현한 원소의 컬렉션을 입력받는 메서드는 주로 정렬, 검색, 최솟값, 최댓값 구하는 방식으로 컬렉션에 담긴 모든 원소가 상호 비교될 수 있어야 한다

<br>

**✏ #05 예제소스 | 재귀적 타입 한정을 이용해 상호 비교할 수 있음을 표현**

```java
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if (c.isEmpty())
        throw new IllegalArgumentException("컬렉션이 비어 있습니다");
    
    E result = null;
    for (E e : c) {
        if (result == null || e.compareTo(result) > 0)
            result = Object.requireNonNull(e);
    }
}
```

>타입 한정인 `<E extends Comparable<E>>`는 **"모든 타입 E는 자신과 비교할 수 있다"**를 표현
>
>컬렉션에 담긴 원소의 자연적 순서를 기준으로 최댓값을 계산하며, 컴파일 오류나 경고가 발생하지 않음

*이 메서드에 빈 컬렉션을 건네면 `IllegalArguemntException`을 던지니, `Optional<E>`를 반환하도록 고치는 편이 좋다*

<br>

**재귀적 타입 한정은 훨씬 복잡해질 가능성은 있지만 잘 일어나지 않는다!**

<br>

---

<br>

### 📌 핵심정리

**제네릭 타입과 마찬가지로, 클라이언트에서 입력 매개변수와 변환값을 명시적으로 형변환해야 하는 메서드보다 제네릭 메서드가 더 안전하며 사용하기도 쉽다**

**타입과 마찬가지로, 메서드도 형변환 없이 사용할 수 있는 편이 좋으며, 많은 경우 그렇게 하려면 제네릭 메서드가 되어야한다**

**역시 타입과 마찬가지로, 형변환을 해줘야 하는 기존 메서드는 제네릭하게 만들자**

**기존 클라이언트는 그대로 둔 채 새로운 사용자의 삶을 훨씬 편하게 만들어줄 것이다**

<br>
