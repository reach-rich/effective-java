## 1. 들어가기

우리는 보통 정형화된 데이터를 표현하기 위해 상수를 사용하곤 합니다.

하지만, 상수를 사용하게 되면 다음과 같은 문제를 일으킬 가능성이 있습니다.

```java
  public static final int CARD_JACK = 11;
  public static final int CARD_QUEEN = 12;
  public static final int CARD_KING = 13;

  public static final int FAMILY_BROTHER_AGE = 11;
  public static final int FAMILY_SISTER_AGE = 13;

  public static void main(String[] args) {
    Card card = new Card(FAMILY_BROTHER_AGE);   // 정상 작동
  }
```

이런 문제점을 어떻게 해결할 수 있을까요?

## 2. 열거 타입

다행히 자바는 앞의 문제점을 말끔히 씻어주면서 여러 장점을 안겨주는 방법으로 열거 타입을 제시했습니다.

열거 타입은 클래스이며 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final로 공개합니다.

따라서 클라이언트는 인스턴스를 직접 생성할 수 없기에 열거 타입 인스턴스는 딱 하나씩만 존재하게 됩니다.

🔸 사용법

```java
  public enum Fruit {
    ORANGE, APPLE, MANGO
  }

  public enum Phone {
    APPLE, SAMSUNG
  }
```

<br>

🔸사용처

필요한 원소를 컴파일 타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용합시다.

ex. 태양계 행성, 요일, 체스 말 등

<br>

🔸장점

  1. 컴파일 시 타입 안전성을 제공한다.

     만약, 메서드의 매개 변수 타입으로 Fruit 열거 타입을 사용했다면

     건네 받은 참조는 null 또는 3가지 타입 중에 하나가 됩니다.

     그러므로 클라이언트는 `bus`와 같이 임의로 값을 지정할 수 없습니다.

      ```java
        public class FruitMarket {
          private Fruit mostPopularFruit;

          public FruitMarket(Fruit fruit) {
            this.mostPopularFruit = fruit
          }
        }

        public static void main(String[] args) {
          FruitMarket market = new FruitMarket(Phone.APPLE);   // 컴파일 오류
        }
      ```

     <br>

  2. 각자 이름공간이 있어 이름이 같은 상수도 사용할 수 있다.

     예를 들어 Fruit 열거 타입과 Phone 열거 타입 각각에 `APPLE`이라는 값이 있어도 공존할 수 있습니다.

     <br>

  3. 열거 타입의 toString 메서드는 출력하기 적합한 문자열을 내어준다.

     상수 필드와 달리 열거 타입은 클래스이기 때문에 메서드를 가질 수 있습니다.

     따라서, toString 메서드를 통해 정보를 출력할 수 있는 적합한 문자열을 내어줄 수 있습니다.

## 3. 열거 타입의 필드와 메서드

또한, 열거 타입에 필드와 메서드를 추가할 수도 있습니다.

예시를 통해 알아봅시다.

```java
  public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS(4.869e+24, 6.052e6),
    EARTH(5.975e+24, 6.378e6),
    MARS(6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN(5.685e+26, 6.027e7),
    URANUS(8.683e+25, 2.556e7),
    NEPTUNE(1.024e+26, 2.447e7);
    
    private final double mass;            // 질량(단위: 킬로그램)
    private final double radius;          // 반지름(단위: 미터)
    private final double surfaceGravity;  // 표면중력(단위: m / s^2)
    
    // 중력상수 (단위: m^3 / kg s^2)
    private static final double G = 6.67300E-11;
    
    // 생성자
    Planet(double mass, double radius) {
      this.mass = mass;
      this.radius = radius;
      this.surfaceGravity = G * mass / (radius * radius);
    }
    
    // 해당 행성에서의 무게 반환
    public double surfaceWeight(double mass) {
      return mass * surfaceGravity;
    }
  }
```

위의 예시는 태양계의 여덟 행성을 나타내는 열거 타입입니다.

각 행성에는 질량과 반지름이 있고, 이 두 속성을 이용해 표면 중력을 계산할 수 있습니다.

따라서, 어떤 객체의 질량이 주어지면 그 객체가 행성 표면에 있을 때의 무게도 계산할 수 있습니다.

```java
  Planet.MERCURY.surfaceWeight(70.0);
```

이렇게 열거 타입에 필드와 메서드를 추가하면 더 많은 데이터를 가진 상수를 만들 수 있습니다.

## 4. 열거 타입 값에 따른 분기 처리

앞의 Planet 예시와 달리 상수마다 동작이 달라져야 하는 상황도 있을 것입니다.

예시를 살펴보겠습니다.

```java
  public enum Operation {
    PLUS, MINUS, TIMES, DIVIDE;

    public double apply(double x, double y) {
      switch(this) {
        case PLUS:    return x + y;
        case MINUS:   return x - y;
        case TIMES:   return x * y;
        case DIVIDE:  return x / y;
      }

      throw new AssertionError("알 수 없는 연산: " + this);
    }
  }
```

위의 예시는 산술 연산 종류에 따라 다른 동작을 하는 열거 타입입니다.

동작에는 아무런 문제가 없지만 새로운 상수를 추가하면 case문을 추가해야하는 문제점이 있습니다.

만약, case 문 작성을 잊어버린다면 컴파일은 되지만, 런타임 시 오류를 내며 프로그램이 종료할 것입니다.

다행히 열거 타입은 더 나은 수단을 제공합니다.

```java
  public enum Operation {
    PLUS  {
      public double apply(double x, double y)   { return x + y; }
    },

    MINUS  {
      public double apply(double x, double y)   { return x - y; }
    },

    TIMES  {
      public double apply(double x, double y)   { return x * y; }
    },

    DIVIDE  {
      public double apply(double x, double y)   { return x / y; }
    };

    public abstract double apply(double x, double y);
  }
```

다음과 같이 열거 타입에 추상 메서드를 선언하고 각 상수에서 재정의 하는 방법입니다.

이를 **"상수별 메서드 구현"**이라고 합니다.

그리고 상수별 메서드 구현을 "상수별 데이터"와도 결합할 수 있습니다.

```java
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

## 5. 전략 열거 타입 패턴

한편, 상수별 메서드 구현은 열거 타입 상수끼리 코드를 공유하기 어렵다는 단점이 있습니다.

예시를 살펴보겠습니다.

```java
  enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;

    private static final int MINS_PER_SHIFT = 8 * 60;

    int pay(int minutesWorked, int payRate) {
      int basePay = minutesWorked * payRate;

      int overtimePay;
      switch (this) {
        case SATURDAY:
        case SUNDAY:
          overtimePay = basePay / 2;
          break;
        default:
          overtimePay = minutesWorked <= MINS_PER_SHIFT ? 0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
      }

      return basePay + overtimePay;
    }
  }
```

위의 예시는 급여명세서에서 쓸 요일을 표현하는 열거 타입입니다.

이 열거 타입은 직원의 일당을 계산해주는 메서드를 가지고 있습니다.

해당 코드는 문제없이 잘 작동합니다.

하지만, 휴가와 같은 새로운 값을 추가하려면 case 문도 함께 처리해줘야 하는 문제점을 가지고 있습니다.

case 문 추가를 자칫 깜빡하면 휴가 기간에 일해도 평일과 똑같은 임금을 받게 되는거죠.

상수별 메서드 구현으로 급여를 정확히 계산하는 방법은 두 가지입니다.

* 잔업수당 계산 코드를 모든 상수에 중복해서 넣는다.

* 잔업수당 계산 코드를 평일용, 주말용으로 나눠 도우미 메서드로 작성한 다음 적절하게 호출한다.

하지만, 두 가지 방법 모두 코드가 장황해져 가독성이 떨어지고 오류 발생 가능성이 높아집니다.

그래서 가장 깔끔한 방법은 잔업수당 **"전략"**을 선택하도록 하는 것입니다.

```java
  enum PayrollDay {
    MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY), THURSDAY(WEEKDAY), FRIDAY(WEEKDAY), SATURDAY(WEEKEND), SUNDAY(WEEKEND);

    private final PayType payType;

    PayrollDay(PayType payType) {
      this.payType = payType;
    }

    int pay(int minutesWorked, int payRate) {
      return payType.pay(minutesWorked, payRate);
    }

    enum PayType {
      WEEKDAY {
        int overtimePay(int minsWorked, int payRate) {
          return minsWorked <= MINS_PER_SHIFT ? 0 : (minsWorked - MINS_PER_SHIFT) * payRate / 2;
        }
      },
      WEEKEND {
        int overtimePay(int minsWorked, int payRate) {
          return minsWorked * payRate / 2;
        }
      };

      abstract int overtimePay(int minsWorked, int payRate);
      private static final int MINS_PER_SHIFT = 8 * 60;

      public int pay(int minsWorked, int payRate) {
        int basePay = minsWorked * payRate;
        return basePay + overtimePay(minsWorked, payRate);
      }
    }
  }
```

잔업수당 계산을 PayType으로 옮기고, PayrollDay 생성자에서 이 중 적당한 것을 선택하게 합니다.

그러면 PayrollDay는 잔업수당 계산을 전략 열거 타입에 위임하기 때문에

switch 문이나 상수별 메서드 구현이 필요없게 됩니다.

이 패턴은 switch 문보다 복잡하지만 더 안전하고 유연합니다.

## 6. switch문을 사용하는 경우

switch 문은 열거 타입의 상수별 동작을 구현하는데 적합하지 않습니다.

하지만 상수별 동작을 혼합해서 넣는 경우, switch문이 좋은 선택이 될 수 있습니다.

```java
  public static Operation inverse(Operation op) {
    switch (op) {
      case PLUS:      return Operation.MINUS;
      case MINUS:     return Operation.PLUS;
      case TIMES:     return Operation.DIVIDE;
      case DIVIDE:    return Operation.TIMES;
    }

    throw new AssertionError("알 수 없는 연산: " + this);
  }
```

## 7. 정리

이번 포스트는 열거 타입에 대해 알아보았습니다.

열거 타입은 읽기 쉽고 안전하며 강력합니다.

대부분의 열거 타입이 명시적 생성자나 메서드 없이 사용하지만

때론, 필요한 경우가 있기 때문에 상황에 맞게 유용하게 사용합시다.
