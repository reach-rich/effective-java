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
- final 필드를 사용하려면  
