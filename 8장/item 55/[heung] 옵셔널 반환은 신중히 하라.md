Effective Java의  55번째 아이템 "옵셔널 반환은 신중히 하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 1. 옵셔널

자바 8 전에는 메서드가 특정 조건에서 값을 반환할 수 없을 때 취할 수 있는 선택지가 두 가지 있었다. 예외를 던지거나 null을 반환하는 것인데, 두 가지 모두 허점이 있다.

예외는 진짜 예외적인 상황에서만 사용해야 하며 예외를 생성할 때 스택 추적 전체를 캡처하므로 비용도 만만치 않다.

null을 반환하면 이런 문제가 생기지 않지만, 클라이언트에서 별도의 null 처리를 해줘야 한다. 그렇지 않으면 언젠가 NullPointerException이 발생할 수 있다.

<br>

**자바 8로 올라가면서 또 하나의 선택지인 옵셔널이 생겼다.** Optional\<T>은 null이 아닌 T 타입 참조를 하나 담거나, 혹은 아무것도 담지 않을 수 있다. 그리고 옵셔널은 원소를 최대 1개 가질 수 있는 '불변' 컬렉션이다.

보통은 T를 반환해야 하지만 특정 조건에서는 아무것도 반환하지 않아야 할 때 T 대신 Optional\<T>를 반환하도록 선언하면 된다. 그러면 유효한 반환값이 없을 때는 빈 결과를 반환하는 메서드가 만들어진다. **옵셔널을 반환하는 메서드는 예외를 던지는 메서드보다 유연하고 사용하기 쉬우며, null을 반환하는 메서드보다 오류 가능성이 작다.**

<br>

예제를 통해 어떻게 사용할 수 있는지 알아보자. 다음은 주어진 컬렉션에서 최댓값을 뽑아주는 메서드다.

```java
public static <E extends Comparable<E>> E max(Collection<E> c) {
  if (c.isEmpty()) {
    throw new IllegalArgumentException("빈 컬렉션");
  }
    
  E result = null;
  for (E e : c) {
    if (result == null || e.compareTo(result) > 0) {
      result = Objects.requireNonNull(e);
    }
  }
  
  return result;
}
```

이 메서드에 빈 컬렉션을 건네면 IllegalArgumentException을 던진다. 옵셔널을 활용해보자.

```java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
  if (c.isEmpty()) {
    return Optional.empty();
  }
    
  E result = null;
  for (E e : c) {
    if (result == null || e.compareTo(result) > 0) {
      result = Objects.requireNonNull(e);
    }
  }
  
  return Optional.of(result);
}
```

이 코드에서는 두 가지 팩터리를 사용했다. 빈 옵셔널은 Optional.empty()로 만들고, 값이 든 옵셔널은 Optional.of(value)로 생성했다. 

Optional.of(value)에 null을 넣으면 NullPointerException을 던지니 주의하자. null 값도 허용하는 옵셔널을 만들려면 Optional.ofNullable(value)를 사용하면 된다. 

옵셔널을 반환하는 메서드에서는 절대 null을 반환하지 말자. 옵셔널을 도입한 취지를 완전히 무시하는 행위다.

<br>

## 2. 옵셔널 활용

null을 반환하거나 예외를 던지는 대신 옵셔널 반환을 선택해야 하는 기준은 무엇일까? 옵셔널은 검사 예외와 취지가 비슷하다. 즉, **반환값이 없을 수도 있음을 API 사용자에게 명확히 알려준다.** 

메서드가 옵셔널을 반환한다면 클라이언트는 값을 받지 못했을 때 취할 행동을 선택해야 한다. 아래 몇 가지 방법이 있다.

```java
// 1) 기본값을 정해둔다.
String lastWordInLexicon = max(words).orElse("단어 없을...");

// 2) 기본값을 정해둔다. Supplier<T>를 인수로 받기 때문에 기본값 설정 비용을 낮출 수 있다.
String lastWordInLexicon = max(words).orElseGet(() -> "단어 없을...");

// 3) 원하는 예외를 던진다.
Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);

// 4) 항상 값이 채워져 있다고 가정한다. (값이 없을 경우 예외 발생)
Element lastNobleGas = max(Elements.NOBLE_GASES).get();
```

이외에 filter, map, flatMap, ifPresent 등 더 특별한 쓰임에 대비한 메서드도 준비되어 있다. 

<br>

스트림을 사용한다면 옵셔널들을 Stream<Optional\<T>>로 받아서, 그중 채워진 옵셔널들에서 값을 뽑아 Stream\<T>에 건네 담아 처리하는 경우가 드물지 않다. 자바 8에서는 다음과 같이 구현할 수 있다. 

```java
streamOfOptionals
  .filter(Optional::isPresent)
  .map(Optional::get)
```

자바 9에서는 Optional에 stream() 메서드가 추가되었다. 이 메서드는 Optional을 Stream으로 변환해주는 어댑처다. 이를 Stream flatMap 메서드와 조합하면 앞의 코드를 다음처럼 명료하게 바꿀 수 있다.

```java
streamOfOptionals
  .flatMap(Optional::stream)
```

<br>

## 3. 주의할 점

반환값으로 옵셔널을 사용한다고 해서 무조건 득이 되는 건 아니다. **컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸면 안 된다.** 빈 Optional\<List\<T>>를 반환하기보다는 빈 List\<T>를 반환하는 게 좋다. 빈 컨테이너를 반환하면 클라이언트에 옵셔널 처리 코드를 넣지 않아도 된다. 

<br>

그렇다면 어떤 경우에 옵셔널을 사용해야 할까? **결과가 없을 수 있으며, 클라이언트가 이 상황을 특별하게 처리해야 한다면 Optional\<T>를 반환하자.** 

그런데 이렇게 하더라도 옵셔널을 반환하는 데는 대가가 따른다. 옵셔널도 엄연히 새로 할당하고 초기화해야 하는 객체이고, 그 안에서 값을 꺼내려면 메서드를 호출해야 하니 한 단계를 더 거치는 셈이다. 그래서 **성능이 중요한 상황에서는 옵셔널이 맞지 않을 수 있다.** 

<br>

박싱된 기본 타입을 담는 옵셔널은 기본 타입 자체보다 무거울 수 밖에 없다. 그래서 자바 API 설계자는 int, long, double 전용 옵셔널 클래스들을 준비해놨다. 바로 OptionalInt, OptionalLong, OptionalDouble이다. 이 옵셔널들도 Optional\<T>가 제공하는 메서드를 거의 다 제공한다. 이렇게 대체제까지 있으니 **박싱된 기본 타입을 담은 옵셔널을 반환하는 일은 없도록 하자.** 

<br>

**옵셔널을 반환값 이외의 용도로 사용하는 것은 대부분 적절하지 않다.** 옵셔널을 컬렉션의 키, 값, 원소나 배열의 원소로 사용하는 게 적절한 상황은 거의 없다. 

그렇다면 옵셔널을 인스턴스 필드에 저장해두는 게 필요할 때가 있을까? 이런 상황은 대부분  필수 필드를 갖는 클래스와, 이를 확장해 선택적 필드를 추가한 하위 클래스를 따로 만들어야 함을 암시하는 좋지 않은 신호다. 

하지만 가끔은 적절항 상황도 있다. 어떤 클래스의 인스턴스 필드 중 필수가 아닌 것들은 값이 없음을 나타낼 방법이 마땅치 않다. 선택적 필드는 게터 메서드들이 옵셔널을 반환하게 해주면 좋을 것이다. 따라서 이러한 경우 필드 자체를 옵셔널로 선언하는 것도 좋은 방법이다. 

<br>

##  4. 핵심 정리

* 값을 반환하지 못할 가능성이 있고, 호출할 때마다 반환값이 없을 가능성을 염두에 둬야하는 메서드라면 옵셔널을 반환해야 할 상황일 수 있다.
* 옵셔널 반환에는 성능 저하가 뒤따르니, 성능에 민감한 메서드라면 null을 반환하거나 예외를 던지는 편이 나을 수 있다.
* 옵셔널을 반환값 이외의 용도로 쓰는 경우는 매우 드물다. 
