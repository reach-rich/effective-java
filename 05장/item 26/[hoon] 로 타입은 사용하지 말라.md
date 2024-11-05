## 1. 들어가기

클래스와 인터페이스 선언에 타입 매개변수가 쓰이면 이를 제네릭 클래스 혹은 제네릭 인터페이스라 하고

제네릭 클래스와 제네릭 인터페이스를 통틀어 제네릭 타입이라고 합니다.

이 제네릭 타입을 정의하면 그에 딸린 Raw 타입도 함께 정의됩니다.

Raw 타입이란, 제네릭 타입에서 타입 매개변수를 전혀 사용하지 않을 때를 말합니다.

예를 들면 `List<E>`의 Raw 타입은 `List`입니다.

그럼, Raw 타입에 대해 자세히 알아보겠습니다.

## 2. Raw 타입

Raw 타입을 사용한다면 어떻게 사용할 수 있을까요?

다음과 같이 타입 매개변수를 작성하지 않으면 Raw 타입으로 사용할 수 있습니다.

```java
  Collection bookList = new ArrayList();
```

하지만 이렇게 작성할 경우 문제가 발생할 수 있습니다.

🔍 문제점

먼저 예시를 살펴봅시다.

```java
  Collection bookList = new ArrayList();
  book.add(new Pencil());
```

해당 코드는 책 컬렉션에 연필이 들어가게 되지만 컴파일러는 아무 오류 없이 컴파일 되고 실행됩니다.

그렇기 때문에 컬렉션 내부를 보는 도중에 오류가 발생하게 됩니다.

이렇게 되면 런타임에 문제를 겪는 코드와 원인을 제공한 코드가 상당히 떨어져 있을 가능성이 커져

코드의 오류를 발견하기 위해서 코드 전체를 훑어봐야 하는 불상사가 생길 수 있습니다.

그래서 Raw 타입 대신 매개변수화 타입을 명시하는 제네릭 타입을 통해 타입의 안전성을 확보해야 합니다.

```java
  Collection<Book> bookList = new ArrayList();
  book.add(new Pencil()); // 컴파일 오류
```

그렇다면 안전성이 확보되지 않는 Raw 타입은 왜 만든 것일까요?

🔍 호환성

초기 Java는 제네릭의 개념이 없었습니다.

시간이 지남에 따라 제네릭의 필요성을 느끼게 되었고 10년이 지난 후에서야 제네릭을 도입했습니다.

하지만, 그동안 제네릭 없이 개발한 코드가 이미 세상을 뒤덮어버렸고

제네릭의 도입은 기존 코드를 모두 수용하면서 새로운 코드와도 맞물려 돌아가게 해야 했고,

그래서 마이그레이션 호환성을 위해 Raw 타입을 지원하게 되었습니다.

## 3. Raw 타입과 매개변수화 타입 비교

Raw 타입의 `List`와 매개변수화 타입의 `List<Object>`의 차이가 무엇일까요?

언뜻 보기에는 별 차이가 없어보이지만, 둘은 큰 차이가 있습니다.

Raw 타입의 `List`는 제네릭 타입에서 완전히 발을 뺀 것이고,

매개변수화 타입의 `List<Object>`는 모든 타입을 허용한다는 의사를 컴파일러에 명확히 전달한 것입니다.

다음의 Raw 타입 사용 예시를 살펴봅시다.

```java
  private static void unsafeAdd(List list, Object o) {
    list.add(o);
  }

  public static void main(String[] args) {
    List<String> stringList = new ArrayList<>();
    unsafeAdd(stringList, Integer.valueOf(42));
    String s = stringList.get(0);
  }
```

해당 코드는 컴파일 오류 없이 잘 실행되지만 

이 프로그램을 이대로 실행하게 되면 Integer를 String으로 변환하려 시도했기 때문에

ClassCastException 예외가 발생합니다.

이번에는 매개변수 타입 사용 예시를 살펴봅시다.

```java
  private static void unsafeAdd(List<Object> list, Object o) {
    list.add(o);
  }

  public static void main(String[] args) {
    List<String> stringList = new ArrayList<>();
    unsafeAdd(stringList, Integer.valueOf(42)); // 컴파일 오류
    String s = stringList.get(0);
  }
```

해당 코드는 컴파일조차 되지 않습니다.

그 이유는 `List<String>`은 `List<Object>`의 하위 타입이 아니기 때문입니다.

💡 결론

> 매개변수화 타입을 사용할 때와 달리 Raw 타입을 사용하면 "타입 안전성"을 잃게 된다.

## 4. 비한정적 와일드카드 타입

Raw 타입은 원소의 타입을 몰라도 되기 때문에 유용하게 보일 수도 있습니다.

앞서 문제가 되었던 Raw 타입 예시를 살펴보겠습니다.

```java
  private static void unsafeAdd(List list, Object o) {
    list.add(o);
  }

  public static void main(String[] args) {
    List<String> stringList = new ArrayList<>();
    unsafeAdd(stringList, Integer.valueOf(42));
    String s = stringList.get(0);
  }
```

해당 코드는 타입 불변식을 훼손해 타입이 안전하지 않는 문제가 있었습니다.

그럼, 원소의 타입을 몰라도 사용할 수 있고 타입 안전성이 보장되는 방법이 있을까요?

이럴 때는 비한정적 와일드카드 타입을 사용하면 됩니다.

비한정적 와일드카드 타입의 사용법은 그저 물음표(?)를 사용하면 됩니다.

위의 예시를 비한정적 와일드카드 타입으로 수정해보겠습니다.

```java
  private static void unsafeAdd(List<?> list, Object o) {
    list.add(o);  // 컴파일 오류
  }

  public static void main(String[] args) {
    List<String> stringList = new ArrayList<>();
    unsafeAdd(stringList, Integer.valueOf(42));
    String s = stringList.get(0);
  }
```

비한정적 와일드카드 타입으로 수정하게 되면 어떠한 원소의 타입을 받을 수 있고

null을 제외한 어떠한 원소도 넣을 수 없기 때문에 타입의 안전성을 보장합니다.

## 5. Raw 타입을 사용해야 하는 경우

앞서 Raw 타입을 사용하지 말아야 한다는 이야기를 했습니다.

하지만 예외는 있습니다.

1. class 리터럴에 사용하는 경우

   자바 명세는 class 리터럴에 매개변수화 타입을 사용하지 못하게 했습니다.

   즉, `List<String>.class`는 허용하지 않고, `List.class`와 같이 Raw 타입으로만 사용할 수 있습니다.

2. instanceof 연산자를 사용하는 경우

   Java의 런타임 시에는 제네릭 타입 정보가 지워집니다.
   
   그러므로 instanceof 연산자는 비한정적 와일드카드 타입 외 매개변수화 타입에는 적용할 수 없습니다.

   ```java
    if(o instanceof Set) {
      Set<?> s = (Set<?>) o;
    }
   ```

## 6. 정리

Raw 타입을 사용하면 런타임 시에 예측하지 못한 예외가 발생할 수 있기 때문에 사용을 지양해야 하지만

class 리터럴 또는 instanceof 연산자를 사용하는 경우에는 Raw 타입을 사용할 수 있습니다.

하지만, 그 나머지의 경우에는 Raw 타입 대신 매개변수화 타입을 사용합시다.