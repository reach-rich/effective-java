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
- 일정 개수의 상수 값을 정의하고 사용하는 타입

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

<br>

- `그런데` 열거 타입에 메서드를 추가하는게 필요한 기능일까?
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
