### 🔍 정수 열거 패턴(int enum pattern) 기법의 단점

**열거 타입은 일정 개수의 상수값을 정의한 다음, 그 외의 값은 허용하지 않는 타입이다**

**✏ #01 예제소스 | 정수 열거 패턴 - 상당히 취약하다!**

 ```java
public static final int APPLE_FUJI		= 0;
public static final int APPLE_PIPPIN		= 1;
public static final int APPLE_GRANNY_SMITH 	= 2;

public static final int ORANGE_NAVEL		= 0;
public static final int ORANGE_TEMPLE		= 1;
public static final int ORANGE_BLOOD		= 0;
 ```

- 타입 안전을 보장할 방법이 없으며 표현력도 좋지 않다
  - 오렌지를 건네야 할 메서드에 사과를 보내고 동등 연산자(==)로 비교하더라도 컴파일러 경고 X
- 정수 열거 패턴을 사용한 프로그램은 깨지기 쉽다
  - 상수의 값이 바뀌면 클라이언트도 반드시 다시 컴파일해야 한다
- 정수 상수는 문자열로 출력하기가 다소 까다롭다
- 정수 대신 문자열 상수는 문자열 값을 그대로 하드코딩하므로 더 안좋다

<br>

---

<br>

### 💡 열거타입(enum type)

```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```

>열거 타입 자체는 클래스이며, 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개
>
>열거 타입은 밖에서 접근할 수 있는 생성자를 제공하지 않으므로 사실상 final
>
>열거 타입은 인스턴스 통제된다

<br>

**열거 타입은 컴파일 타입 안전성을 제공한다**

- 다른 타입의 값을 넘기려 하면 컴파일 오류가 발생

**열거 타입에는 각자의 이름공간이 있어서 이름이 같은 상수도 평화롭게 공존한다**

- 열거 타입에 새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일할 필요 없다

**열거 타입의 toString 메서드는 출력하기에 적합한 문자열을 내어준다**

**열거 타입에는 임의의 메서드나 필드를 추가할 수 있고 임의의 인터페이스를 구현할 수도 있다**

- Object 메서드들을 높은 품질로 구현
- Comparable과 Serializable을 구현, 그 직렬화 형태도 웬만큼 변형을 가해도 문제 없음

<br>

---

<br>

### ❓ 열거타입에 메서드나 필드를 추가

**✏ #02-1 예제소스 | 데이터와 메서드를 갖는 열거 타입**

```java
public enum Plant {
    MERCURY(3.302e+23, 2.439e6),
    VENUS(4.869e+24, 6.052e6),
    EARTH(5.975e+24, 6.378e6),
    ...
    
    private final double mass;
    private final double radius;
    private final double surfaceGravity;
    
    private static final double G = 6.67300E-11;
    
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }
    
    public double mass()			{ return mass; }
    public double radius()			{ return radius; }
    public double surfaceGravity	{ return surfaceGravity; }
    
    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;
    }
}
```

>- 태양계를 예로 들었을 때, 각 행성의 질량이나 반지름이 있고 이를 통해 표면중력을 계산할 수 있음
>- Planet의 생성자에서 표면중력을 계산해 저장한 이유는 최적화 때문
>
>- Planet 열거 타입은 단순하지만 놀랍도록 강력함

**그저 상수 모임일 뿐인 열거 타입이지만, 고차원의 추상 개념 하나를 완벽히 표현해낼 수 있다**

<br>

**✏ #02-2 예제소스 | 어떤 객체의 지구에서의 무게를 입력받아 행성에서의 무게를 출력하는 코드**

```java
public class WeightTable {
    public static void main(String[] args) {
        double earthWeight = Double.parseDouble(args[0]);
        double mass = earthWeight / Planet.EARTH.surfaceGravity();
        for (Planet p : Planet.value())
            System.out.printf("%s에서의 무게는 %f이다. %n", p, p.surfaceWeight(mass));
    }
}
```

>- 열거 타입은 자신안에 정의된 상수들의 값을 배열에 담아 반환하는 정적 메서드 values를 제공
>- toString 메서드는 상수 이름을 문자열로 반환 (재정의 가능)

<br>

- 열거 타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장
- 열거 타입은 기본적으로 모든 필드는 final이어야한다
  (필드를 public 선언해도 되지만, private으로 두고 별도 public 접근자 메서드를 두는게 낫다)
- 널리 쓰이는 열거 타입은 톱레벨 클래스로 만들고, 특정 톱레벨 클래스에서만 쓰인다면 해당 클래스의 멤버 클래스로 만든다

<br>

---

<br>

### 열거 타입에서 상수하나를 제거하면?

- 클라이언트에서는 프로그램을 다시 컴파일하면 제거된 상수를 참조하는 줄에서 디버깅에 유용한 오류를 담은 컴파일 오류 발생
- 클라이언트를 다시 컴파일하지 않으면 같은 줄에서 런타임에 예외 발생
- 정수 열거 패턴에서는 기대할 수 없는 바람직한 대응

<br>

---

<br>

### 상수마다 동작이 달라져야하는 상황이라면?

**switch문을 이용한 분기**

...

>깨지기 쉬운 코드
>
>상수를 추가하면 해당 case문도 추가해야 한다

<br>

**apply 추상 메서드 선언 후 재정의**

**✏ #04 예제소스 | 상수별 메서드 구현을 활용한 열거 타입**

```java
public enum Operation {
    PLUS {public double apply(double x, double y) {return x + y; }},
    MINUS {public double apply(double x, double y) {return x - y; }},
    TIMES {public double apply(double x, double y) {return x * y; }},
    DIVIDE {public double apply(double x, double y) {return x / y; }},
    
    public abstract double apply(double x, double y);
}
```

>새로운 상수를 추가할 때 apply도 재정의 필요하다 (하지 않으면 컴파일 오류로 알려줌)
>
>상수별 메서드 구현을 상수별 데이터와 결합할 수도 있다

<br>

---

<br>

### 📚 전략 열거 타입 패턴

...

<br>

---

<br>

### 📌 핵심정리

**열거 타입은 확실히 정수 상수보다 뛰어나다**

**더 읽기 쉽고 안전하고 강력하다**

**대다수 열거 타입이 명시적 생성자나 메서드 없이 쓰이지만, 각 상수를 특정 데이터와 연결짓거나 상수마다 다르게 동작하게 할 때는 필요하다**

**드물게는 하나의 메서드가 상수별로 다 르게 동작해야 할 때도 있다**

**이런 열거 타입에서는 switch문 대신 상수별 메서드 구현을 사용하자**

**열거 타입 상수 일부가 같은 동작을 공유한다면 전략 열거 타입 패턴을 사용하자**

