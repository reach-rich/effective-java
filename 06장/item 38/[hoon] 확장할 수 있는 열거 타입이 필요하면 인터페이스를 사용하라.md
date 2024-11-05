## 1. 들어가기

타입 안전 열거 패턴은 클래스를 이용해 private 생성자로 만들어 최초 정의된 객체만 참조하는 패턴으로

다음과 같이 사용할 수 있습니다.

```java
  public class Symbol {
    private final String type;

    private Symbol(String type){
      this.type = type;
    }

    public String toString(){
      return type;
    }
  }
```

```java
  public static final Symbol Terminal = new Symbol("Terminal");
  public static final Symbol Process = new Symbol("Process");
  public static final Symbol Decision = new Symbol("Decision");
```

하지만, 우리가 배운 열거 타입은 단 한 가지 예외를 제외하고 타입 안전 열거 패턴보다 우수한데

어떤 예외가 있는지 알아보겠습니다.

## 2. 열거 타입의 한 가지 예외

타입 안전 열거 패턴은 열거한 값들을 그대로 가져와서 값을 추가해 확장할 수 있습니다.

하지만, 열거 타입은 그럴 수 없습니다.

사실 대부분 상황에서 열거 타입을 확장하는 것은 좋지 않지만,

연산 코드와 같은 예시는 확장 연산을 추가할 수 있도록 열어줘야할 때가 있습니다.

다행히 열거 타입으로 확장할 수 있는 방법이 있습니다.

## 3. 열거 타입의 한 가지 예외 해결법

열거 타입 자체는 확장할 수 없지만, 인터페이스를 통해 확장을 가능하도록 만들 수 있습니다.

```java
  public interface Operation {
    double apply(double x, double y);
  }
```

```java
/* 기본 열거 타입 */
  public enum BasicOperation implements Operation {
    PLUS("+") {
      public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
      public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
      public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
      public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    BasicOperation(String symbol) {
      this.symbol = symbol;
    }

    @Override public String toString() {
      return symbol;
    }
  }
```

연산 타입을 Operation 인터페이스로 지정해서 사용하면

Operation 인터페이스를 구현한 또 다른 열거 타입을 정의할 수 있습니다.

```java
/* 새로 확장한 열거 타입 */
  public enum ExtendedOperation implements Operation {
    EXP("^") {
      public double apply(double x, double y) {
        return Math.pow(x, y);
      }
    },
    REMAINDER("%") {
      public double apply(double x, double y) {
        return x % y;
      }
    };

    private final String symbol;

    ExtendedOperation(String symbol) {
      this.symbol = symbol;
    }

    @Override public String toString() {
      return symbol;
    }
  }
```

그리고 그 새로운 열거 타입은 BasicOperation 열거 타입을 대체할 수 있습니다.

새로운 열거 타입을 사용하기 위해서는 Operation 인터페이스만을 허용해야 하는데 

두 가지 방법 중 하나를 선택할 수 있습니다.

1. 한정적 타입 토큰 역할을 하는 Class 리터럴을 사용한 방법

   ```java
    static <T extends Enum<T> & Operation> void test(Class<T> opEnumType, double x, double y) {
      for (Operation op : opEnumType.getEnumConstants())
        System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
    }
   ```

2. 한정적 와일드카드 타입을 넘기는 방법

   ```java
    static void test(Collection<? extends Operation> opSet, double x, double y) {
      for (Operation op : opSet)
        System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
    }
   ```

## 4. 정리

타입 안전 열거 패턴과 열거 타입을 비교했을 때 대부분 열거 타입이 우수하지만

그 자체로는 확장이 불가능한 단점이 있었습니다.

이를 해결하기 위해서는 인터페이스와 해당 인터페이스를 구현하는 열거 타입을 통해

확장과 같은 효과를 낼 수 있습니다.