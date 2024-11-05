## 1. 들어가기

여러분은 Java에서 객체를 정렬할 때 어떤 방법으로 하시나요?

Java에서는 객체를 정렬하기 위해 보통 두 가지의 정렬 방법 중 하나를 선택합니다.

바로, Comparable 인터페이스 또는 Comparator 인터페이스를 사용하는 것인데요.

이번 포스트에서는 이 두 가지 방법 중 Comparable 인터페이스를 다뤄보도록 하겠습니다.

## 2. Comparable 인터페이스

Comparable이 어떻게 동작하는지 알아보기 위해 Comparable 인터페이스의 내부를 살펴보겠습니다.

```java
/* Comparable 인터페이스 */
  package java.lang;
  import java.util.*;

  public interface Comparable<T> {
      public int compareTo(T o);
  }
```

Comparable 인터페이스는 내부에 compareTo라는 메서드 하나가 존재합니다.

compareTo는 객체의 순서를 비교하는 메서드로 이 객체가 주어진 객체보다 작으면 음의 정수를,

같으면 0을, 크면 양의 정수를 반환합니다.

## 3. compareTo 메서드 일반 규약

compareTo 메서드도 일반 규약이 존재하는데 함께 살펴보겠습니다.

* ```sgn(x.compareTo(y)) == -sgn(y.compareTo(x))```

  반사성에 대한 규약으로 두 객체 참조의 순서를 바꿔 비교해도 예상한 결과가 나와야한다는 의미입니다.

* ```(x.compareTo(y) > 0 && y.compareTo(z) > 0) 이면``` ```x.compareTo(z) > 0```

  추이성에 대한 규약으로 첫 번째가 두 번째보다 크고, 두 번째가 세 번째보다 크면

  첫 번째는 무조건 세 번째보다 커야한다는 의미입니다.

* ```x.compareTo(y) == 0 이면``` ```sgn(x.compareTo(z)) == sgn(y.compareTo(z)```

  대칭성에 대한 규약으로 크기가 같은 객체들은 어떤 객체와 비교해도 항상 같아야한다는 의미입니다.

* ```(x.compareTo(y)) == 0) == (x.equals(y))```

  이는 필수는 아니지만 권고에 해당되는 규약입니다.

  compareTo 메서드로 수행한 동치성 테스트 결과는 equals 결과와 같아야 한다는 것입니다.

  BigDecimal 클래스와 같이 compareTo와 equals의 결과가 다른 클래스도 있으나
  
  이 클래스의 객체를 정렬된 Collection에 넣으면 정의된 동작과 엇박자를 낼 수도 있습니다.

## 4. compareTo 메서드 작성 요령

그렇다면 compareTo 메서드는 어떻게 작성하면 될까요?

compareTo 메서드의 작성 요령은 equals 메서드와 비슷합니다.

단, 몇 가지 차이점만 주의하면 되는데 한번 알아보겠습니다.

1. 입력 인수의 타입을 확인하거나 형변환할 필요가 없다.

    Comparable 인터페이스는 타입을 인수로 받는 제네릭 인터페이스입니다.
  
    그러므로 compareTo 메서드의 인수 타입은 컴파일 타임에 정해지기 때문에

    인수의 타입이 잘못됐다면 컴파일 오류가 발생할 것입니다.

2. 각 필드가 동치인지를 비교하는게 아닌 그 순서를 비교한다.

    객체 참조 필드를 비교하려면 compareTo 메서드를 재귀적으로 호출합니다.

    하지만, Comparable을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야한다면,

    Comparator를 대신 사용합니다.

## 5. Comparator를 통한 compareTo 메서드 작성법

Comparator는 직접 만드는 방법과 Java가 제공하는 방법 2가지 방법이 있습니다.

* 직접 만드는 방법

```java
  import java.util.Comparator;

  public class Person implements Comparable<Person> {
    String name;
    Age age;

    @Override
    public int compareTo(Person p) {
      return Age.compare(age, p.age);
    }
  }
```

* Java가 제공하는 방법

```java
  public class Person implements Comparable<Person> {
    String name;
    int age;

    @Override
    public int compareTo(Person p) {
      return Integer.compare(age, p.age);
    }
  }
```

자바 8에서는 Comparator 인터페이스가 일련의 비교자 생성 메서드와 팀을 꾸려

메서드 연쇄 방식으로 비교자를 생성할 수 있게 되었습니다.

그리고 이를 compareTo 메서드를 구현하는데 멋지게 활용될 수 있습니다.

```java
  import java.util.Comparator;

  public class Person implements Comparable<Person> {
    String name;
    int age;

    private static final Comparator<Person> COMPARATOR = Comparator.comparingInt((Person p) -> p.age)
                                                                    .thenComparing(p -> p.name);

    @Override
    public int compareTo(Person p) {
        return COMPARATOR.compare(this, p);
    }

    // 생성자, toString 생략
  }
```

## 6. 잘못된 예시

흔히들 값의 차를 기준으로 비교자를 작성하는 경우가 있습니다.

바로, 아래와 같은 예시처럼 말이죠.

```java
  static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
      return o1.hashCode() - o2.hashCode();
    }
  }
```

이 방식은 정수 Overflow를 발생시킬 수 있기 때문에 사용을 지양해야 합니다.

그럼 어떤 방식으로 작성할 수 있을까요?

위에서 알아본 정적 compare 메서드 또는 비교자 생성 메서드를 이용할 수 있겠네요.

* 정적 compare 메서드 사용

  ```java
    static Comparator<Object> hashCodeOrder = new Comparator<>() {
      public int compare(Object o1, Object o2) {
        return Integer.compare(o1.hashCode(), o2.hashCode());
      }
    }
  ```

* 비교자 생성 메서드 사용

  ```java
    static Comparator<Object> hashCodeOrder = Comparator.comparingInt(o -> o.hashCode());
  ```

## 7. 정리

이번 포스트에서는 객체를 정렬하기 위한 방법 중 하나인 Comparable 인터페이스에 대해서 알아보았습니다.

Comparable 인터페이스의 내부에 있는 compareTo 메서드는 객체의 정렬 방법을 정할 수 있습니다.

compareTo 메서드를 작성할 때는 일반 규약과 작성 요령에 맞춰 작성해야 하고,

필드의 값을 비교할 때는 정적 compare 메서드나 비교자 생성 메서드로 작성해봅시다.