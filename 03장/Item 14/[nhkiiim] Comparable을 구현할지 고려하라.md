# Comparable을 구현할지 고려하라

### 1. Comparable 인터페이스의 유일무이한 메서드! compareTo
- Object의 메서드가 아님 

- equals와 비슷하다 -> compareTo는 단순 동치성 비교와 순서까지 비교 가능
- Comparable을 구현했다는건,,? 그 클래스에는 자연적인 순서가 있다는 것!!

```java
Arrays.sort(a);
```

- Comparable을 구현하면 이처럼 간편하게 정렬 가능

<br>

- 검색, 극단값 계산, 자동 정렬되는 컬렉션도 쉽게 관리 가능

```java
public class WordLiist {
  public static void main(String[] args){
    Set<String> s = new TreeSet<>();
    Collections.addAll(s, args);
    System.out.println(s);
  }
}
```

- 좁쌀 노력 코끼리 효과! 별점 오점 Comparable

- 얘는 짱이니까 알파벳, 숫자, 연대 같이 순서가 명확한 클래스에서는 Comparable 인터페이스를 구현하도록 하자!

#
### 2. compareTo 일반 규약

    매개 변수 인스턴스와 자신의 인스턴스를 비교하도록 해야한다. 또한 자신의 인스턴스가 매개변수보다 작으면 음수, 같으면 0, 크면 양수를 반환한다.
    
    모든 x,y에 대해 sgn(x.compareTo(y)) == -sgn(y.compareTo(x))여야 한다.
    
    x.compareTo(y) > 0 && y.compareTo(z) > 0 이면 x.compareTo(z) > 0이다. (추이성)
    
    x.compareTo(y) == 0 이면 모든 z에 대해 sgn(x.compareTo(z)) == sgn(y.compareTo(z)) 이다.
    

- 비교를 활용하는 클래스의 예 : TreeSet, TreeMap, Collections(검색과 정렬 알고리즘을 활용하는 유틸리티 클래스), Arrays
 
- 위의 compareTo의 마지막 규약은 필수는 아니지만 꼭 지키는게 좋을거다
- Collections, Set, Map의 구현체에서 예상과 다르게 동작할 수 있기 때문!

- 이 인터페이스들은 equals 메서드의 규약을 따른다고 되어 있지만 정렬된 컬렉션들은 동치성을 비교할 때 equals 대신 CompareTo를 사용하기 때문에 지키는게 좋음

<br>

- compareTo와 equals가 일관되지 않는 BigDecimal 클래스 
- 빈 HashSet 인스턴스를 생성한 다음 new BigDecimal("1.0")과 new BigDecimal("1.00")을 차례로 추가
- 이 두 BigDecimal은 equals 메서드로 비교하면 서로 다르기 때문에 HashSet은 원소를 2개 갖게 됨
- 하지만 HashSet 대신 TreeSet을 사용하면 원소를 하나만 갖게 됨
- compareTo 메서드로 비교하면 두 BigDecimal 인스턴스가 똑같기 때문!!

#
### 3. 주의사항

- 기존 클래스를 확장한 구체 클래스에서 새로운 값 컴포넌트(필드)를 추가했다면 compareTo 규약을 지킬 방법이 없음.. 
- 이 주의사항은 equals에도 적용!! 우회법은 역시 equals에서의 우회법과 같음(기억이..^0^) 
- Comparable을 구현한 클래스를 확장해 값 컴포넌트를 추가하고 싶다 -> 확장하는 대신 독립된 클래스를 만들기

- 이 클래스에 원래 클래스의 인스턴스를 가리키는 필드를 두고 내부 인스턴스를 반환하는 '뷰' 메서드를 제공하자
- 그럼 바깥 클래스에 우리가 원하는 compareTo 메서드를 구현해 넣을 수 있음
- 클라이언트는 필요에 따라 바깥 클래스의 인스턴스를 필드 안에 담긴 원래 클래스의 인스턴스로 다룰 수도 있을 것!!
 
<br>

- 기본 타입 필드를 비교할 때는 ">", "<" 이런 연산자를 사용하지 말자
- 자바7부터 추가된 정적 메서드인 Double.compare, Float.compare 사용을 권한다고~!

- compareTo에서 관계 연산자를 활용하는 방식은 거추장스럽고 오류를 유발하니 이제는 추천하지 않아~! (나 완전 거추장 됐다)

```java
public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
  public int compareTo(CaseInsensitiveString cis) {
    return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
  }
}
```

#
### 4. 필드가 여러개일 때 비교하기
- 클래스에 핵심 필드가 여러개라면 어느 것을 먼저 비교할지가 중요함
- 가장 핵심적인 필드부터 비교를 하자

```java
public int compareTo(PhoneNumber pn) {
    int result = Short.compare(areaCode, pn.areaCode);
    if (result == 0)  {
        result = Short.compare(prefix, pn.prefix);
        if (result == 0)
            result = Short.compare(lineNum, pn.lineNum);
    }
    return result;
}
```

- 자바 8부터는 Comparator 인터페이스가 비교자 생성 메서드와 함께 연쇄 방식으로 비교자를 생성할 수 있음 
- 이 비교자들을 Comparable 인터페이스가 원하는 compareTo 메서드를 구현하는데 활용할 수 있음 다만 성능이 좀 느려질 뿐~!!

```java
private static final Comparator<PhoneNumber> COMPARATOR =
        Comparator.comparingInt((PhoneNumber pn) -> pn.areaCode)
                .thenComparingInt(pn -> pn.prefix)
                .thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
    return COMPARATOR.compare(this, pn);
}
```
- long과 double용으로는 thenComparingLong / thenComparingDouble
- short처럼 더 작은 정수 타입에는 int용 메서드를 사용
- float은 double용 메서드를 사용

- Comparator는 자바의 모든 숫자용 기본 타입에 대한 비교자 생성 메서드를 지원해주는 멋쟁이~!!



#
### 5. 이러지마 제발

- 값의 차를 기준으로 첫 번째 값이 두 번째 값보다 작으면 음수를, 두 값이 같으면 0을, 첫 번째 값이 크면 양수를 반환하는 compareTo나 compare 메서드를 만들면 안됨 (이런!)
- 정수 오버플로를 일으키거나 IEEE 754 부동소수점 계산 방식에 따른 오류를 낼 수 있음

- 이번에 구현한 코드보다 월등히 빠르지도 않음 (ㅇㅋ)

```java
static Comparator<Object> hashCodeOrder = new Comparator<Object>() {
    public int compare(Object o1, Object o2) {
        return o1.hashCode() - o2.hashCode();
    }
}
```

<br>

`이제는 이 두가지 방법 중 하나를 쓰자~`

```java
static Comparator<Object> hashCodeOrder = new Comparator<Object>() {
    public int compare(Object o1, Object o2) {
        return Integer.compare(o1.hashCode(), o2.hashCode());
    }
}
```

```java
static Comparator<Object> hashCodeOrder = 
  Comparator.comparingInt(o -> o.hashCode());
```
 
 
 
 
