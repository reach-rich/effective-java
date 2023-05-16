## 1. 들어가기

Java 8 이전에는 메서드가 값을 반환할 수 없을 때 예외를 던지거나 null을 반환하곤 했습니다.

하지만 두 방법 모두 허점이 있습니다.

예외는 진짜 예외적인 상황에서 사용해야 하고, 예외를 생성할 때의 비용 또한 만만치 않습니다.

그리고 null을 반환하면 클라이언트에서 별도의 null 처리 코드를 추가해야 하는 문제가 있습니다.

Java 8부터는 `Optional` 이라는 또 하나의 선택지가 생겼습니다.

## 2. Optional

`Optional<T>`은 T타입 참조를 하나 담거나, 혹은 아무것도 담지 않을 수 있습니다.

`Optional`은 다음과 같이 사용할 수 있습니다.

```java
  public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
    if (c.isEmpty())
      return Optional.empty();

    E result = null;
    for (E e : c) {
      if (result == null || e.compareTo(result) > 0)
        result = Objects.requireNonNull(e);
    }

    return Optional.of(result);
  }
```

빈 `Optional`은 `Optional.empty()`로 만들고, 값이 든 `Optional`은 `Optional.of(value)`로 만들 수 있습니다.

여기서 value에 `null`을 넣으면 `NullPointerException`이 발생하니 주의해야 합니다.

그렇다고 `null`을 그대로 반환하면 `Optional` 사용 취지를 무시하는 것이므로

`Optional` 값에 `null`을 허용하는 `Optional.ofNullable(value)`을 사용합시다.

## 3. Optional의 활용

* Optional의 기본값 지정

  ```java
    String lastWordInLexicon = optional(words).orElse("단어 없음");
  ```

* Optional의 기본값 지정(기본값 비용이 큰 경우)

  ```java
    String lastWordInLexicon = optional(words).orElseGet(() -> { getDefaultWord() });
  ```

* Optional으로 예외 발생

  ```java
    Toy myToy = optional(toys).orElseThrow(TemperTantrumException::new);
  ```

* 항상 값이 채워져 있다고 가정

  ```java
    Element lastNobleGas = optional(Elements.NOBLE_GASES).get();
  ```

* Stream 활용

  ```java
    streamOfOptionals
      .filter(Optional::isPresent)
      .map(Optional::get)
  ```

* Java 9부터 추가된 stream() 메서드

  ```java
    streamOfOptionals
      .flatMap(Optional::stream)
  ```

## 4. Optional 주의사항

1. Collection, Stream, Array, Optional 같은 컨테이너 타입은 Optional로 감싸면 안된다.

   그 이유는 아래와 같이 애매한 경우가 발생할 수 있기 때문입니다.

   ```java
    List<String> emptyList = new ArrayList<>();
    Optional<List<String>> list = Optional.of(emptyList);
    if(list.isPresent()){
      System.out.println("빈 리스트이지만 사용 가능 객체??"); 
    }else {
      System.out.println("빈 리스트이므로 사용 불가능 객체??");
    }
   ```

2. 박싱된 기본타입을 담은 Optional을 반환하는 일은 없도록 해야한다.

   박싱된 기본 타입을 담는 Optional은 값을 두겹이나 감싸기에 기본 타입보다 무거울 수 밖에 없습니다.

   그래서 Java API 설계자는 int, long, double 전용 Optional 클래스를 준비해놨습니다.

   ```java
    OptionalInt optionalInt = OptionalInt.of(100);
    OptionalDouble optionalDouble = OptionalDouble.of(100.0);
   ```