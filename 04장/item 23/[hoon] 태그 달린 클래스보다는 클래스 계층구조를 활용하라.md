## 1. 들어가기

개발을 하다보면 두 가지 의미를 표현하는 클래스를 종종 볼 수 있습니다.

이 두 가지 의미를 표현하는 클래스는 둘 중 현재 의미하는 클래스를 태그 값으로 명시하는데

이를 '태그 달린 클래스'라고 합니다.

## 2. 태그 달린 클래스

두 가지 의미를 표현하는 클래스인 태그 달린 클래스는 다음과 같은 형태를 가집니다.

```java
  class Figure {
    enum Shape { RECTANGLE, CIRCLE };

    // 태그 필드 (현재 모양)
    final Shape shape;

    // 사각형일 때 사용하는 필드
    double width;
    double height;

    // 원일 때 사용하는 필드
    double radius;

    // 원용 생성자
    Figure(double radius) {
      shape = Shape.CIRCLE;
      this.radius = radius;
    }

    // 사각형용 생성자
    Figure(double width, double height) {
      shape = Shape.RECTANGLE;
      this.width = width;
      this.height = height;
    }

    // 면적 구하는 메서드
    double area() {
      switch(shape) {
        case RECTANGLE:
          return width * height;
        case CIRCLE:
          return Math.PI * (radius * radius);
        default:
          throw new AssertionError(shape);
      }
    }
  }
```

## 3. 태그 달린 클래스의 문제점

위 예시의 문제점이 무엇일까요?

우선, 가장 먼저 보이는 열거 타입, 태그 필드, switch 문 등 쓸데없는 코드가 많습니다.

또한, 여러 구현이 한 클래스에 혼합되어 있어 가독성도 좋지 않고

사각형 인스턴스는 사각형을 구현하는데 필요없는 원용 필드를 가지기 때문에 메모리 낭비도 됩니다.

마지막으로 인스턴스 타입만으로 현재 나타내는 의미를 알 수 없습니다.

즉, 태그 달린 클래스는 장황하고, 오류를 내기 쉽고 비효율적입니다.

그럼 이를 어떻게 해결할 수 있을까요?

## 4. 클래스 계층구조

태그 달린 클래스의 문제점은 클래스 계층구조 변경으로 해결할 수 있습니다.

가장 먼저 계층 구조의 root가 될 추상 클래스를 정의하고

태그 값에 따라 동작이 달라지는 메서드를 추상 메서드로 선언하며

모든 하위 클래스에서 공통으로 사용하는 데이터 필드도 함께 선언합니다.

그리고 이 추상 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의합니다.

다음은 위의 내용을 종합한 예시입니다.

```java
/* 모양 추상(root) 클래스 */
  abstract class Figure {
    abstract double area();
  }

/* 원 클래스 */
  class Circle extends Figure {
    final double radius;

    Circle(double radius) { this.radius = radius; }

    @Override
    double area() {
      return Math.PI * (radius * radius);
    }
  }

/* 사각형 클래스 */
  class Rectangle extends Figure {
    final double width;
    final double height;

    Rectangle(double width, double height) {
      this.width = width;
      this.height = height;
    }

    @Override
    double area() {
      return width * height;
    }
  }
```

## 5. 클래스 계층구조의 장점

클래스 계층구조는 다음과 같은 장점이 있습니다.

* 런타임 오류가 발생할 일이 없다.

  태그 달린 클래스의 경우, case 문 작성을 잊어도 컴파일러가 확인하지 못합니다.

  그래서 런타임 시에 오류가 발생할 가능성이 있는 반면

  클래스 계층구조의 경우, 컴파일러는 각 클래스의 생성자가 모든 필드를 초기화하고
  
  추상 메서드를 모두 구현했는지 확인할 수 있기 때문에 런타임 오류 발생 가능성이 없습니다.

* 확장성

  태그 달린 클래스의 경우, 또 하나의 모양을 추가하기 위해서는
  
  열거 타입과 필드를 추가하고 switch의 case 문을 수정해야 합니다.

  하지만 클래스 계층 구조의 경우, 루트 클래스는 건드리지 않고 쉽게 모양 클래스를 추가할 수 있습니다.

* 타입 별로 구분할 수 있다.

  태그 달린 클래스의 경우, 한 타입만을 가지므로 원과 사각형을 타입만으로 구별하지 못합니다.

  하지만 클래스 계층구조의 경우, 각 모양마다 다른 타입을 가지기 때문에
  
  변수의 의미를 명시 또는 제한하거나 매개변수로도 받을 수 있습니다.

## 6. 정리

이번 포스트는 태그 달린 클래스와 클래스 계층구조를 비교해보았습니다.

태그 달린 클래스는 장황하고 오류를 내기 쉽고, 비효율적인 반면

클래스 계층구조는 런타임 오류 가능성이 없고 확장성이 있으며 타입별로 구분할 수 있는 장점이 있습니다.

그렇기 때문에 새로운 클래스를 작성하는 데 태그 필드가 등장한다면

태그를 없애고 계층구조로 대체하는 방법을 생각해봅시다.