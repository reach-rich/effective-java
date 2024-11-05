# 태그 달린 클래스보다는 클래스 계층구조를 활용하라

### 1. 태그달린 클래스

```java
public class Figure {
    enum Shape {RECTANGLE, CIRCLE}

    //태그 필드 - 현재 모양을 나타낸다.
    private Shape shape;

    // 다음 필드들은 모양이 사각형일 때만 사용.
    private double length;
    private double width;

    // 다음 필드들은 모양이 원일 때만 싸용.
    private double radius;

    //원용 생성자
    public Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    //사각형용 생성자
    public Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    private double area() {
        switch (shape) {
            case RECTANGLE:
                return length + width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```

- 태그 달린 클래스 -> 객체 구분하려고 변수로 이름 달아주는 클래스 (김나현 뇌피셜)
- 태그달린 클래스는 단점이 한가득 ! 

- 열거 타임 선언, 태그필드, 스위치문 등 쓸데없는 코드가 많아짐
- 여러 구현이 한 클래스에 구현되어 있어 가독성도 쓰레기!
- 다른 의미를 가진 애들을 한곳에 모아두니까 메모리도 많이 필요함
- final 필드를 사용하려면 해당 의미에 쓰이지 않는 필드까지 생성자에 초기화 해야함
- 아무튼 한마디로 태그 달린 클래스는 장황하고 오류내기 쉽고 비효율적임

#
### 2. 클래스 계층 구조 사용하기
- 자바는 객체지향 언어로 다양한 의미의 객체를 표현하는 다양한 수단을 제공
- 태그 달린 클래스는 계층구조를 어설프게 따라한 것 뿐임 ㅋㅋ

<br>

- __태그달린 클래스를 클래스 계층 구조로 바꾸기__

1) 계층구조의 root가 되리 추상 클래스 정의
2) 태그 값에 따라 달라지는 메서드를 루트 클래스의 추상 메서드로 선언
3) 태그 값에 상관 없이 모든 클래스에서 공통으로 사용하는 메서드를 루트 클래스에 일반 메서드로 추가
4) 모든 하위 클래스에서 공통으로 사용하는 데이터 필드들도 전부 루트 클래스로 올리기
5) 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의

```java
abstract class Figure{
    abstract double area;
}

class Circle extends Figure{
    final double radius;
    Circle(double radius) {this.radius = radius}
    
    @Override
    double area(){
        return Math.PI * (radius * radius);
    }
}

class Rectangle extends Figure{
    final double length;
    final double width;
    
    Rectangle (double length, double width) {
        this.length = length;
        this.width = width;
    }
    
    @Override
    double area(){
        return length * width;
    }
}
```
