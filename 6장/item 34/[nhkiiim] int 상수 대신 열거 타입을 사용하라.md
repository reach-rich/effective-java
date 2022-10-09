# int 상수 대신 열거 타입을 사용하라

### 1. 정수 열거 패턴

- 자바에서 열거 타입을 지원하기 전에는 정수 상수들을 선언해 사용하곤 함

```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;

public static final int ORANGE_NAVEL = 0
public static final int ORANGE_TEMPLE = 1;
```

- __정수 열거 패턴에는 단점이 많음__

  - 타입 안전을 보장할 방법이 없으며 표현력이 좋지 않음
  
  - 평범한 상수를 나열한거라 컴파일하면 값이 클라이언트 파일에 그대로 새겨짐 (상수의 값이 바뀌면 다시 컴파일 해야함)
  - 단순 숫자 값을 출력해 문자열으로 출력하기 어려움
  - 상수 묶음의 상수들이 총 몇개인지도 알기 어려움

- 문자열 상수를 사용하는 변형 패턴도 있으나 더 별로임

#
### 2. 열거타입
- __일정 개수의 상수 값을 정의하고 사용하는 타입__

```java
public enum Apple {FUJI, PIPPIN, RANNY_SMITH}
public enum Orange {NAVEL, TEMPLE, BLOOD}
```

- 겉보기에는 C, C# 과 같은 다른 언어의 열거 타입과 비슷해 보이지만 자바의 열거타입은 다르다!
- 자바의 열거 타입은 `완전한 형태의 클래스`

- 상수 하나당 자시느이 인스턴스를 만들어 public static final 필드로 공개
- 열거 타입은 생성자를 재공하지 않아 사실상 final! (인스턴스들이 딱 하나씩만 존재)
- 열거 타입은 싱글턴을 일반화한 형태라고 볼 수 있음

<br>

- 열거 타입은 컴파일 타임 타입 안전성을 제공
- 열거 타입은 각자의 이름 공간이 있어 이름이 같은 상수도 공존 가능!
- 새로운 상수를 추가 하거나 순서를 바꿔도 컴파일 하지 않아도 됨 (필드의 이름만 공개되어 클라이언트에 컴파일되어 각인되지 않기 때문!)

- 열거 타입의 toString 메서드로 문자열을 출력할 수도 있음
- 열거 타입에 임의의 메서드를 추가하거나 인터페이스를 구현할 수 도 있음
- 열거 타입을 선언한 후 그 패키지에서만 유용한 기능은 private이나 package-private 메서드로 구현한다!
- 널리 쓰이는 열거 타입은 톱레벨 클래스로 만들기도 한다
  - 톱레벨 클래스
    - 중첩되지 않은 클래스
    - 다른 클래스나 인터페이스 내부에 선언되지 않은 클래스를 의미   


#

🧐 `그런데` __열거 타입에 메서드를 추가하는게 필요한 기능일까?__
  - 상수와 연관된 데이터를 상수 자체에 내장시키고 싶다면! 유용하게 사용가능
    - ex) APPLE 에 색을 알려주거나 이미지를 반환하는 메서드 추가 가능

```java
//데이터와 메서드를 갖는 열거 타입 예시
public enum Planet {
   MERCURY(3.302e+23, 2.439e6),
   VENUS (4.869e+24, 6.052e6),
   EARTH (5.975e+24, 6.378e6),
   
   private final double mass;		// 질량(단위 : 킬로그램)
   private final double redius; 	// 반지름(단위: 미터)
   private final double surfaceGravity; 	// 표면중력(단위: m / s^2)
   
   // 중력상수(단위: m^3 / kg s^2)
   private static final double G = 6.67300E-11;
   
   // 생성자
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
- 이렇게 거대한 열거 타입을 만드는 것도 어렵지 않음

- 열거 타입 상수 각각을 특정 데이터와 연결짓기 위해 상성자에서 데이터를 받아 인스턴스 필드에 저장하면 됨
- 열거 타입은 기본적으로 불변이라 __모든 필드는 final__ 이어야함!

<br>

- Planet 열거타입은 단순하지만 강력하다!
- 아래와 같이 8행성에서의 무게를 출력하는 일을 짧은 코드로 작성할 수 있다!

```java
public class WeightTable {
   public static void main(String[] args) {
      double earthWeight = Double.parseDouble(args[0]);
      double mass = earthWeight / Planet.EARTH.surfaceGravity();
      
      for (Planet p : Planet.value())
         System.out.printf("%s에서의 무게는 %f이다.%n", p, p.surfaceWeight(mass));
   }
}
```

#

🧐 __행성 하나를 빼고싶어서 상수 하나를 삭제해야 한다면?__ 

- WeightTable과 같은 경우는 수정할 필요도 없다

- 제거된 상수를 참조하는 클라이언트가 있을 경우에는 디버깅에 유용한 컴파일 오류가 발생할 것 (아주 바람직한 대응)


#
### 3. 이대로 만족하는가 (상수별 동작)

🤪 __상수마다 동작이 달라져야하는 경우__

```java
public enum Operation {
   PLUS, MINUS, TIMES, DIVIDE;
   
   // 상수가 뜻하는 연산을 수행한다.
   public double apply(double x, double y) {
      switch(this) {
         case PLUS: return x + y;
         case MINUS: return x - y;
         case TIMES: return x * y;
         case DIVIDE: return x / y;
      }
      throw new AssertionError("알 수 없는 연산: " + this);
   }
}
```
- 그리 예쁘진 않지만 동작은 한다 (하지만 깨지기 쉬운 코드)
- 새로운 상수가 추가되면 case 문도 추가해야함 (추가 안하면 런타임 오류 발생)

<br>

- 열거 타입은 상수별로 다르게 동작하는 더 나은 수단을 제공
- 열거 타입에 apply라는 추상 메서드를 선언하고 재정의 하는 방법

`상수별 메서드 구현`

```java
public enum Operation {
   PLUS { public double apply(double x, double y) { return x+y; }},
   MINUS { public double apply(double x, double y) { return x-y;}},
   TIMES { public double apply(double x, double y) { return x*y;}},
   DIVIDE { public double appply(double x, double y) { return x/y;}};
   
   public abstract double apply(double x, double y);
}
```
<br>

`연산 기호까지 예쁘게 출력하기`

```java
public enum Operation {
   PLUS("+") {
      public double apply(double x, double y) { return x+y; }
   },
   MINUS("-") {
      public double apply(double x, double y) { return x-y; }
   },
   TIMES("*") {
      public double apply(double x, double y) { return x*y; }
   },
   DIVIDE("/") {
      public double apply(double x, double y) { return x/y; }
   };
   
   private final String symbol;
   
   Operation(String symbol) { this.symbol = symbol; }
   
   @Override
   public String toString() { return symbol; }
   public abstract double apply(double x, double y);
}
```

- 열거 타입은 상수의 이름을 받아 해당 상수를 출력해주는 valueOf(String) 메서드가 자동 생성

- 그리고 toString 메서드를 재정의하려면, 해당 문자열을 상수로 변환하주는 fromString도 같이 고려해보자

```java
private static final Map<String, Operation> stringToEnum = 
      Stream.of(values()).collect(
         toMap(Object::toString, e -> e));

// 지정한 문자열에 해당하는 Operation을 (존재한다면) 반환한다.
public static Optional<Operation> fromString(String symbol) {
   return Optional.ofNullable(stringToEnum.get(symbol));
}
```

#
🤪 __상수별 메서드 구현의 단점__

```java
//급여 명세서 요일 표현 열거타입
enum PayrollDay {
   MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;
   
   private static final int MINS_PER_SHIFT = 8 * 60;
   
   int pay(int minutesWorked, int payRate) {
      int basePay = minutesWorked * payRate;
      
      int overtimePay;
      switch(this) {
         case SATURDAY: case SUNDAY: // 주말
            overtimePay = basePay / 2;
            break;
         default: // 주중
            overtimePay = minutesWorked <= MINS_PER_SHIFT ?
               0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
      }
      
      return basePay + overtimePay;
   }
}
```
- 상수끼리 코드를 공유하기 어렵다는 단점
- 휴가와 같은 새로운 값을 열거 타입에 추가하면 case도 꼭 넣어줘야함

#
🤪 __전략 열거 타입으로 해결__

```java
enum PayrollDay {
   MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY(PayType.WEEKEND), SUNDAY(PayType.WEEKEND);
   
   private final PayType payType;
   
   PayrollDay(PayType payType) { this.payType = payType; }
   
   int pay(int minutesWorked, int payRate) {
      return payType.pay(minutesWorked, payRate);
   }
   
   // 전략 열거 타입
   enum PayType {
      WEEKDAY {
         int overtimePay(int minsWorked, int payRate) { 
            return minsWorked <= MINS_PER_SHIFT ? 0 :
               (minsWorked - MINS_PER_SHIFT) * payRate / 2;
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
- 수당 게산을 private 중첩 열거 타입을 사용하는 것
- switch문보다 복잡하지만 안전!

- 결론 : switch문은 기존 열거 타입에 상수별 동작을 혼합해 넣을 땐 좋을 수도 있지만 상수별로 동작을 구현하는데에는 적합하지 않음!

#

🤪 __이런 것도 있다네__

```java
//반대 연산을 반환하는 메서드
public static Operation inverse(Operation op) {
   switch(op) {
      case PLUS: return Operation.MINUS;
      case MINUS: return Operation.PLUS;
      case TIMES: return Operation.DIVIDE;
      case DIVIDE: return Operation.TIMES;
      
      default: throw new AssertionError("알 수 없는 연산: " + op);
   }
}
```

- 그래서 언제 스위치 쓰고 언제 안쓰라는건지 조꿈 헷갈림

#
### 4. 마무리

- 열거 타입의 성능은 정수 상수와 별반 다르지 않음
- 열거 타입을 메모리에 올리는 공간과 초기화 하는 시간이 들긴 하지만 체감 될 정도는 아님!

- __필요한 원소를 컴파일 타임에 다 알 수 있는 상수 집합인 경우에는 열거 타입을 사용하자__
- 열거 타입에 정의된 상수 개수가 영원히 고장 불변일 필요도 없다~
