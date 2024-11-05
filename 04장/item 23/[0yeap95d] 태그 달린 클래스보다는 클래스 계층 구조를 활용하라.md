### 🏷 태그 달린 클래스

**: 두 가지 이상의 의미를 표현할 수 있으며, 그중 현재 표현하는 의미를 태그 값으로 알려주는 클래스**

<br>

**✏ #01 예제소스 | 태그 달린 클래스 - 클래스 계층구조보다 훨씬 나쁘다**

```java
Class Figure {
    enum Shape { RECTANGLE, CIRCLE };
    
    // 태그 필드 - 현재 모양을 나타낸다.
    final Shape shape;
    
    // 다음 필드들은 모양이 사각형(RECTANGLE)일 때만 쓰인다.
    double length;
    double width;
    
    // 다음 필드는 모양이 원(CIRCLE)일 때만 쓰인다.
    double radius;
    
    // 원용 생성자
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }
    
    double area() {
		switch(shape) {
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

> 원과 사각형을 표현할 수 있는 클래스

<br>

---

<br>

### ❌ 태그 달린 클래스 단점

- 열거 타입 선언, 태그 필드, switch문 등 쓸데없는 코드가 많다
- 여러 구현이 한 클래스에 혼합되어 가독성이 나쁘다
- 다른 의미를 위한 코드도 언제나 함께 하니 메모리도 많이 사용한다
- 필드들을 `final`로 선언하려면 해당 의미에 쓰이지 않는 필드들까지 생성자에 초기화해야 한다
- 생성자가 태그 필드를 설정하고 해당 의미에 쓰이는 데이터 필드들을 초기화하는 데 컴파일러가 도와줄 수 있는건 별로 없다
- 엉뚱한 필드를 초기화해도 런타임에야 문제가 드러난다
- 또 다른 의미를 추가하려면 코드를 수정해야한다
- 인스턴스의 타입만으로는 현재 나타내는 의미를 알 길이 없다

**즉, 태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율적이다**

<br>

---

<br>

### 🛠 태그 달린 클래스를 계층구조로 바꾸는 방법

1. 계층구조의 루트(root)가 될 추상 클래스를 정의하고, 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 선언한다
2. 태그 값에 상관없이 동작이 일정한 메서드들을 루트 클래스에 일반 메서드로 추가한다
3. 모든 하위 클래스에서 공통으로 사용하는 데이터 필드들도 전부 루트 클래스로 올린다
4. 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의한다

<br>

**✏ #02 예제소스 | 태그 달린 클래스를 클래스 계층구조로 변환**

```java
abstract class Figure {
    abstract double area();
}

class Circle extends Figure {
    final double radius;
    
    Circle(double radius) { this.radius = radius; }
    
    @Override double area() { return Math.PI * (radius * radius); }
}

class Rectangle extends Figure {
    final double length;
    final double width;
    
    Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }
    
    @Override double area() { return length * width; }
}
```

<br>

---

<br>

### ✔ 계층구조 장점

- 계층구조는 위에서 언급한 태그 달린 클래스의 단점을 해결 가능하다

- 간결하고 명확하며 쓸데없는 코드도 사라진다

- 타입 사이의 자연스러운 계층 관계를 반영할 수 있어서 유연성은 물론 컴파일타임 타입 검사 능력도 높여준다

<br>

---

<br>

### 📌 핵심정리

**태그 달린 클래스를 써야하는 상황은 거의 없다**

**새로운 클래스를 작성하는 데 태그 필드가 등장한다면 태그를 없애고 계층구조로 대체하는 방법을 생각해보자**

**기존 클래스가 태그 필드를 사용하고 있따면 계층구조로 리팩터링하는 걸 고민해보자**

<br>
