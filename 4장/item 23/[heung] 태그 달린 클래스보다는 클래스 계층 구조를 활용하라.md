Effective Java의 스물세 번째 아이템 "태그 달린 클래스보다는 클래스 계층구조를 활용하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 1. 태그 달린 클래스

어떤 클래스는 두 가지 이상의 의미를 표현할 수 있으며, 그중 현재 표현하는 의미를 태그 값으로 알려준다. 예를 들어 다음과 같이 원과 사각형을 표현할 수 있는 Figure 클래스가 있다.

```java
class Figure {
  enum Shape { RECTANGLE, CIRCLE };
  
  final Shape shape; // 태그 필드 - 현재 모양을 나타낸다.
  
  // 사각형일 때만 쓰인다.
  double length;
  double width;
  
  // 원일 때만 쓰인다.
  double radius;
  
  // 원 생성자
  Figure(double radius) {
    shape = Shape.CIRCLE;
    this.radius = radius;
  }
  
  // 사각형 생성자
  Figure(double length, double width) {
    shape = Shape.RECTANGLE;
    this.length = length;
    this.width = width;
  }
  
  double area() {
    switch (shape) {
      case RECTANGLE:
        return length * width;
      case CIRCLE:
        return Math.PI * (radius * radius);
      default:
        throw new AssertionError(shape);
    }
  }
}
```

이러한 태크 달린 클래스는 많은 단점을 가지고 있다.

- 열거 타입 선언, 태그 필드, switch 문 등 쓸데없는 코드가 많다.
- 여러 구현이 한 클래스에 혼합돼 있어서 가독성이 나쁘다.
- 다른 의미를 위한 코드도 언제나 함께 하니 메모리도 많이 사용한다.
- 필드를 final로 선언하려면 해당 의미에 쓰이지 않는 필드들까지 생성자에서 초기화해야 한다.
- 엉뚱한 필드를 초기화해도 런타임에야 문제가 드러난다.
- 또 다른 의미를 추가하려면 코드를 수정해야 한다.
- 인스턴스 타입만으로는 현재 나타내는 의미를 알 길이 전혀 없다.

**한마디로 태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율적이다.**

<br>

## 2. 클래스 계층구조

다행히 자바와 같은 객체 지향 언어는 타입 하나로 다양한 의미의 객체를 표현하는 훨씬 나은 수단을 제공한다. 바로 클래스 계층구조를 활용하는 서브타이핑(subtyping)이다. 

**다음은 태그 달린 클래스를 클래스 계층 구조로 바꾸는 방법이다.**

1. 계층구조의 루트가 될 추상 클래스를 정의하고, 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 선언한다.
2. 태그 값에 상관없이 동작이 일정한 메서드들을 루트 클래스에 일반 메서드로 추가한다. 모든 하위 클래스에서 공통으로 사용하는 데이터 필드들도 전부 루트 클래스로 올린다.
3. 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의한다. 각 하위 클래스에는 각자의 의미에 해당하는 데이터 필드들을 넣는다.
4. 루트 클래스가 정의한 추상 메서드를 각자의 의미에 맞게 구현한다.

<br>

다음은 Figure 클래스를 클래스 계층구조 방식으로 구현한 코드다.

```java
abstract class Figure {
  abstract double area();
}

class Circle extends Figure {
  final double radius;
  
  Circle(double radius) {
    this.radius = radius;
  }
  
  @Override
  double area() {
    return Math.PI * (radius * radius);
  }
}
  
class Rectangle extends Figure {
  final double length;
  final double width;

  Rectangle(double length, double width) {
    this.length = length;
    this.width = width;
  }

  @Override
  double area() {
    return length * width;
  }
}
```

**클래스 계층구조는 태그 달린 클래스의 단점을 모두 날려버린다.**

- 쓸데없는 코드가 모두 사라져 간결하고 명확하다.
- 각 의미를 독립된 클래스에 담아 관련 없던 데이터 필드를 모두 제거했다. 살아남은 필드들은 모두 final이다.
- 각 클래스의 생성자가 모든 필드를 남김없이 초기화하고 추상 메서드를 모두 구현했는지 컴파일러가 확인해준다.
- 실수로 빼먹은 case 문 때문에 런타임 오류가 발생할 일이 없다.
- 루트 클래스의 코드를 건드리지 않고도 다른 프로그래머들이 독립적으로 계층구조를 확장하고 함께 사용할 수 있다.
- 타입이 의미별로 따로 존재하니 변수의 의미를 명시하거나 제한할 수 있고, 또 특정 의미만 매개변수로 받을 수 있다.
- 타입 사이의 자연스러운 계층 관계를 반영할 수 있어서 유연성은 물론 컴파일타임 타입 검사 능력을 높여준다.

<br>

참고로 이번 아이템의 예에서는 접근자 메서드 없이 필드를 직접 노출했다. 이는 단지 코드를 단순하게 하려는 의도였고, 만약 공개할 클래스라면 이렇게 설계하는 것은 좋지 않다.

<br>

## 3. 핵심 정리

- 새로운 클래스를 작성하는 데 태그 필드가 등장한다면 태그를 없애고 계층구조로 대체하는 방법을 생각해보자.
- 기존 클래스가 태그 필드를 사용하고 있다면 계층구조로 리팩터링하는 걸 고민해보자.

<br>

## 4. Related Posts

- [접근자 메서드 (Item 16)](https://heung27.github.io/posts/item-16-public-%ED%81%B4%EB%9E%98%EC%8A%A4%EC%97%90%EC%84%9C%EB%8A%94-public-%ED%95%84%EB%93%9C%EA%B0%80-%EC%95%84%EB%8B%8C-%EC%A0%91%EA%B7%BC%EC%9E%90-%EB%A9%94%EC%84%9C%EB%93%9C%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC/)
