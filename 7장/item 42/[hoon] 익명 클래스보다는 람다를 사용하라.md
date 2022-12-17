## 1. 들어가기

예전에는 자바에서 함수 타입을 표현할 때, 추상 메서드를 하나만 담은 인터페이스를 사용했습니다.

이런 유형의 인터페이스를 인스턴스화한 것을 함수 객체라고 하는데

이 함수 객체를 만들기 위해서 보통 익명 클래스를 사용했습니다.

```java
  Collections.sort(words, new Comparator<String>() {
    public int compare(String s1, String s2) {
      return Integer.compare(s1.length(), s2.length());
    }
  })
```

하지만 익명 클래스 방식은 코드가 길어지는 단점이 있기 때문에 

Java 8에서 도입된 함수형 프로그래밍에 적합하지 않았습니다.

그럼, Java 8에서는 이러한 함수 객체를 어떻게 익명 클래스 대신 생성할 수 있을까요?

## 2. Lambda

Java 8에서는 하나의 추상 메서드를 가진 인터페이스가 특별한 의미를 인정받게 되고,

람다(Lambda Expression)라는 것을 통해 함수 객체를 만들 수 있게 됩니다.

람다는 익명 클래스와 개념은 비슷하지만 코드는 훨씬 간결합니다.

위의 예시를 람다로 바꿔보면 다음과 같습니다.

```java
  Collections.sort(words, (s1, s2) -> Integer.compare(s1.length, s2.length()));
```

자세히 보면 매개변수의 반환 타입 `Comparator<String>`과 `String`이 보이지 않는데

그 이유는 우리 대신 컴파일러가 타입을 추론해 주었기 때문입니다.

상황에 따라 컴파일러가 타입을 결정해주지 못하는 경우가 있는데,

그럴 때는 프로그래머가 직접 명시해줘야 합니다.

하지만, 타입 추론 규칙은 매우 복잡하고 어렵기 때문에 가장 쉽게 적용할 수 있는 방법은

타입을 명시해야 코드가 더 명확할 때만 제외하고, 람다의 모든 매개변수 타입을 생략합니다.

만약, 컴파일러가 타입을 알 수 없다는 오류를 내면 그 때, 해당 타입을 명시하도록 합니다.

## 3. 비교자 생성 메서드

람다 자리에 비교자 생성 메서드를 사용하면 코드를 더욱 간결하게 만들 수 있습니다.

```java
  Collections.sort(words, comparingInt(String::length));
```

추가로, Java 8의 List 인터페이스에 추가된 sort 메서드를 이용하면 더욱 간결하게 만들 수 있습니다.

```java
  words.sort(comparingInt(String::length));
```

## 4. Lambda 사용 예시

[Item 34](../item34)에서 사용했던 Operation 열거 타입을 람다로 바꿀 수 있습니다.

```java
/* 기존 Operation 코드 */
  public enum Operation {
    PLUS("+")  {
      public double apply(double x, double y)   { return x + y; }
    },

    MINUS("-")  {
      public double apply(double x, double y)   { return x - y; }
    },

    TIMES("*")  {
      public double apply(double x, double y)   { return x * y; }
    },

    DIVIDE("/")  {
      public double apply(double x, double y)   { return x / y; }
    };

    public abstract double apply(double x, double y);

    private final String symbol;

    Operation(String symbol)  { this.symbol = symbol; }
  }
```

람다를 이용하면 열거 타입의 인스턴스 필드를 이용하여 상수별로 동작 코드를 쉽게 구현할 수 있습니다.

단순히 각 열거 타입 상수의 동작을 람다로 구현해 생성자에 넘기고

생성자는 이 람다를 인스턴스 필드로 저장하기 때문에 동일하게 apply 메서드를 호출하기만 하면 됩니다.

```java
/* 람다를 사용한 코드 */
  public enum Operation {
    PLUS("+", (x, y) -> x + y),
    MINUS("-", (x, y) -> x - y),
    TIMES("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);

    private final String symbol;
    private final DoubleBinaryOperator op;

    public double apply(double x, double y) {
      return op.applyAsDouble(x, y);
    }

    Operation(String symbol, DoubleBinaryOperator op)  { 
      this.symbol = symbol;
      this.op = op;
    }
  }
```

## 5. Lambda를 지양해야 하는 경우

지금까지만 보면 람다를 무조건 사용해야 하는 것으로 보이지만 람다를 지양해야 하는 경우도 있습니다.

1. 문서화를 해야하는 경우

   람다는 문서화를 할 수 없기 때문에 코드 자체로 동작이 명확하게 설명되는 경우에만 사용합니다.

   <br>

2. 가독성이 떨어지는 경우

   람다는 세 줄 이상 넘어가면 가독성이 심하게 나빠집니다.

   <br>

3. 직렬화하는 경우

   이것은 익명 클래스도 마찬가지인데 직렬화 형태가 구현별로 다를 수 있기 때문에

   익명 클래스, 람다 둘다 직렬화하는 일은 극히 삼가야 합니다.

## 6. Lambda 대신 익명 클래스를 쓰는 경우

다음과 같이 람다를 적용할 수 없을 때는 익명 클래스를 사용해야 합니다.

1. 추상 클래스의 인스턴스를 만드는 경우

   추상 클래스의 인스턴스를 만들 때는 람다를 사용할 수 없습니다.

   대신, 익명 클래스로 대체할 수 있습니다.

   <br>

2. 자신을 참조해야 하는 경우

   람다에서 this 키워드는 바깥 인스턴스를 가리키는 반면,

   익명 클래스에서의 this는 자기 자신을 가리킵니다.

   그래서 함수 객체가 자신을 참조해야 하는 경우에는 반드시 익명 클래스를 사용해야 합니다.

## 7. 정리

이번 포스트는 java 8에서 도입된 람다에 대해 알아보았습니다.

람다는 익명 클래스의 간결하지 않은 단점을 보완하고 타입 추론이 가능한 것을 볼 수 있었습니다.

하지만, 람다가 모든 경우에 적합한 것은 아니기 때문에

경우에 따라 익명 클래스와 람다를 적절하게 사용해야 합니다.