Effective Java의 서른네 번째 아이템 "int 상수 대신 열거 타입을 사용하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 1. 정수 열거 패턴

자바에서 열거 타입을 지원하기 전에는 다음 코드처럼 정수 상수를 한 묶음 선언해서 사용하곤 했다.

```java
public static final int APPLE_FUJI =         0;
public static final int APPLE_PIPPIN =       1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL =  0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD =  2;
```

위와 같은 방식을 정수 열거 패턴이라 하는데, 이 기법에는 단점이 많다. 

* 타입 안전을 보장할 방법이 없으며 표현력도 좋지 않다.
* 상수의 값이 바뀌면 클라이언트도 반드시 다시 컴파일해야 한다.
* 정수 상수는 문자열로 출력하기 다소 까다롭다.
* 같은 정수 열거 그룹에 속한 모든 상수를 한 바퀴 순회하는 방법이 마땅치 않다.

<br>

## 2. 열거 타입

열거 타입은 열거 패턴의 단점을 완전히 해소하며 여러 장점을 가지고 있다.

```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```

자바 열거 타입을 뒷받침하는 아이디어는 단순하다. 얼거 타입 자체는 클래스이며, 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개한다. 열거 타입은 밖에서 접근할 수 있는 생성자를 제공하지 않으므로 사실상 final이다. 따라서 클라이언트가 인스턴스를 직접 생성하거나 확장할 수 없으니 열거 타입 선언으로 만들어진 인스턴스들은 딱 하나씩만 존재함이 보장된다. 싱글턴은 원소가 하나뿐인 열거 타입이라 할 수 있고, 거꾸로 열거 타입은 싱글턴을 일반화한 형태라고 볼 수 있다.

### 장점

* 열거 타입은 컴파일타임 타입 안정성을 제공한다.
* 열거 타입에는 각자의 이름공간이 있어서 이름이 같은 상수도 평화롭게 공존한다.
* 열거 타입에 새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일하지 않아도 된다.
* 열거 타입의 toString 메서드는 출력하기에 적합한 문자열을 내어준다.
* 열거 타입에는 임의의 메서드나 필드를 추가할 수 있고 임의의 인터페이스를 구현하게 할 수도 있다.

<br>

### 예제 1. Planet

태양계의 여덟 행성은 거대한 열거 타입을 설명하기에 좋은 예다. 각 행성에는 질량과 반지름이 있고, 이 두 속성을 이용해 표면중력을 계산할 수 있다.

```java
public enum Planet {
  MERCURY(3.302e+23, 2.439e6),
  VENUS  (4.869e+24, 6.052e6),
  EARTH  (5.975e+24, 6.378e6),
  MARS   (6.419e+23, 3.393e6),
  JUPITER(1.899e+27, 7.149e7),
  SATURN (5.685e+26, 6.027e7),
  URANUS (8.683e+25, 2.556e7),
  NEPTUNE(1.024e+26, 2.477e7);
  
  private final double mass;           // 질량
  private final double radius;         // 반지름
  private final double surfaceGravity; // 표면중력
  
  private static final double G = 6.67300E-11; // 중력 상수
  
  Planet(double mass, double radius) {
    this.mass = mass;
    this.radius = radius;
    surfaceGravity = G * mass / (radius * radius);
  }
  
  public double mass() { return mass; }
  public double radius() { return radius; }
  public double surfaceGravity() { return surfaceGravity; }
  
  public double surfaceWeight(double mass) {
    return mass * surfaceGravity; // F = ma
  }
}
```

열거 타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다. 열거 타입은 근본적으로 불변이라 모든 필드는 final이어야 한다. 필드를 public으로 선언해도 되지만, private으로 두고 별도의 public 접근자 메서드를 두는 게 낫다.

열거 타입은 자신 안에 정의된 상수들의 값을 배열에 담아 반환하는 정적 메서드인 values를 제공한다. 값들은 선언된 순서로 저장된다. 각 열거 타입 값의 toString 메서드는 상수 이름을 문자열로 반환한다. 기본 toString이 제공하는 이름이 내키지 않으면 원하는 대로 재정의하면 된다.

열거 타입에서 상수 하나를 제거할 경우, 제거된 상수를 참조하는 클라이언트는 어떻게 될까? 클라이언트 프로그램을 다시 컴파일하면 제거된 상수를 참조하는 줄에서 디버깅에 유용한 메시지를 담은 컴파일 오류가 발생할 것이다. 클라이언트를 다시 컴파일하지 않으면 런타임에, 역시 같은 줄에서 유용한 예외가 발생할 것이다. 정수 열거 패턴에서는 기대할 수 없는 가장 바람직한 대응이라 볼 수 있다.

<br>

### 예제 2. Operation

Planet 상수들은 서로 다른 데이터와 연결되는 데 그쳤지만, 한 걸음 더 나아가 상수마다 동작이 달라져야 하는 상황도 있을 것이다. 예를 들어 사칙연산 계산기의 연산 종류를 열거 타입으로 선언하고, 실제 연산까지 열거 타입 상수가 직접 수행했으면 한다고 해보자.

```java
public enum Operation {
  PLUS, MINUS, TIMES, DIVIDE;
  
  public double apply(double x, double y) {
    switch (this) {
      case PLUS: return x + y;
      case MINUS: return x - y;
      case TIMES: return x * y;
      case DIVIDE: return x / y;
    }
    throw new AssertionError("알 수 없는 연산: " + this);
  }
}
```

switch 문을 이용해 상수의 값에 따라 분기하는 방법이다. 동작은 하지만 그리 좋은 코드는 아니다. 마지막의 throw 문은 실제로는 도달할 일이 없지만 기술적으로는 도달할 수 있기 때문에 생략하면 컴파일조차 되지 않는다. 더 나쁜 점은 깨지기 쉬운 코드라는 사실이다. 새로운 상수를 추가하면 해당 case 문도 추가해야 한다. 혹시라도 깜빡한다면, 컴파일은 되지만 런타임 오류를 내며 프로그램이 종료된다. 

열거 타입은 상수별로 다르게 동작하는 코드를 구현하는 더 나은 수단을 제공한다. 열거 타입에 apply라는 추상 메서드를 선언하고 각 상수별 클래스 몸체, 즉 각 상수에서 자신에 맞게 재정의하는 방법이다. 이를 상수별 메서드 구현이라 한다.

```java
public enum Operation {
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
  
  Operation(String symbol) { this.symbol = symbol; }
  
  @Override
  public String toString() { return symbol; }
  
  public abstract double apply(double x, double y);
}
```

apply 메서드가 상수 선언 바로 옆에 붙어 있으니 새로운 상수를 추가할 때 apply도 재정의해야 한다는 사실을 깜빡하기는 어려울 것이다. 그뿐만 아니라 apply가 추상 메서드이므로 재정의하지 않았다면 컴파일 오류로 알려준다.

<br>

### 예제 3. PayrollDay

상수별 메서드 구현에는 열거 타입 상수끼리 코드를 공유하기 어렵다는 단점이 있다. 급여명세서에 쓸 요일을 표현하는 열거 타입을 예로 생각해보자. 이 열거 타입은 직원의 시간당 기본 임금과 그날 일한 시간이 주어지면 일당을 계산해주는 메서드를 갖고 있다. 주중에 오버타임이 발생하면 잔업수당이 주어지고, 주말에는 무조건 잔업수당이 주어진다.

```java
enum PayrollDay {
  MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY), 
  THURSDAY(WEEKDAY), FRIDAY(WEEKDAY),
  SATURDAY(WEEKDAY), SUNDAY(WEEKDAY);
  
  private final PayType payType;
  
  PayrollDay(PayType payType) { this.payType = payType; }
  
  int pay(int minutesWorked, int payRate) {
    return payType.pay(minutesWorked, payRate);
  }
  
  enum PayType {
    WEEKDAY {
      int overtimePay(int minsWorked, int payRate) {
        return minsWorked <= MINS_PER_SHIFT ? 0 :
        (minuWorked - MIN_PER_SHIFT) * payRate / 2;
      }
    },
    WEEKEND {
      int overtimePay(int minsWorked, int payRate) {
        return minsWorked * payRate / 2;
      }
    };
    
    abstract int overtimePay(int mins, int payRate);
    
    private static final int MINS_PER_SHIFT = 8 * 60;
    
    int pay(int minsWorked, int payRate) {
      int basePay = minsWorked * payRate;
      return basePay + overtimePay(minsWorked, payRate);
    }
  }
}
```

switch 문을 이용하면 case 문을 날짜별로 두어 간결하게 계산할 수 있지만, 관리 관점에서는 위험한 코드가 된다. 휴가와 같은 새로운 값을 열거 타입에 추가하려면 그 값을 처리하는 case 문을 잊지 말고 쌍으로 넣어줘야 하는 것이다.

가장 깔끔한 방법은 새로운 상수를 추가할 때 잔업수당 '전략'을 선택하도록 하는 것이다. 잔업수당 계산을 private 중첩 열거 타입(PayType)으로 옮기고 PayrollDay 열거 타입의 생성자에서 이중 적당한 것을 선택한다. 그러면 PayrollDay 열거 타입은 잔업수당 계산을 그 전략 열거 타입에 위임하여, switch 문이나 상수별 메서드 구현이 필요 없게 된다. 이 패턴은 복잡하지만 더 안전하고 유연하다.

위처럼 switch 문은 열거 타입의 상수별 동작을 구현하는 데 적합하지 않다. 하지만 기존 열거 타입에 상수별 동작을 혼합해 넣을 때는 switch 문이 좋은 선택이 될 수 있다.

<br>

## 3. 핵심 정리

* 열거 타입은 상수와 비교하여 더 읽기 쉽고 안전하고 강력하다.
* 필요한 원소를 컴파일타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자.
* 대다수 열거 타입이 명시적 생성자나 메서드 없이 쓰이지만, 각 상수를 특정 데이터와 연결짓거나 상수마다 다르게 동작하게 할 때는 필요하다.
* 드물게는 하나의 메서드가 상수별로 다르게 동작해야 할 때도 있다. 이런 열거 타입에서는 switch 문 대신 상수별 메서드 구현을 사용하자.
* 열거 타입에 정의된 상수 개수가 영원히 고정 불변일 필요는 없다.
* 열거 타입 상수 일부가 같은 동작을 공유한다면 전략 열거 타입 패턴을 사용하자.

<br>

## 4. Related Posts

* 싱글턴 (Item 3)
* 불변 (Item 17)
* 접근자 메서드 (Item 16)
* 접근 권한 (Item 15)
* 멤버 클래스 (Item 24)
