## 1. 들어가기

public 클래스에서 public 필드를 사용하면 어떻게 될까요?

```java
  public class Point {
    public double x;
    public double y;
  }
```

Point 클래스는 public으로 선언된 공용 API인데 필드의 접근 범위 또한 public으로 선언하게 된다면

클라이언트에서 직접 필드를 수정할 수 있습니다.

그렇게 된다면 캡슐화의 이점을 제공하지 못하게 되고, 객체의 오용을 방지할 수 없게 됩니다.

그럼 어떻게 수정하면 좋을까요?

## 2. 해결 방법

이런 경우에는 필드를 모두 private으로 바꾸고 public 접근자를 추가하는 방법을 사용할 수 있습니다.

```java
  public class Point {
    private double x;
    private double y;

    public double getX() { return x; }
    public double getY() { return y; }

    public void setX() { this.x = x; }
    public void setY() { this.y = y; }
  }
```

만약, public 클래스가 필드를 public으로 공개하게 된다면

이를 사용하는 클라이언트가 있을 수 있고 내부 표현방식을 마음대로 바꿀 수 없기 때문에

필드는 private으로 감추고 public 접근자를 통해서만 데이터를 주고 받을 수 있게 합니다.

하지만, package-private이나 private 중첩 클래스라면 데이터 필드를 public으로 노출한다해도

패키지 바깥에서 사용할 수 없기 때문에 public 접근자를 사용할 필요가 없습니다.

## 3. public 필드를 노출한 사례

자바 플랫폼 라이브러리에도 필드를 public으로 노출시킨 사례가 있습니다.

이 클래스들을 흉내 내지 말고, 타산지석으로 삼읍시다.

* Point 클래스

```java
  public class Point extends Point2D implements java.io.Serializable {
    public int x;
    public int y;
  }
```

* Dimension 클래스

```java
  public class Dimension extends Dimension2D implements java.io.Serializable {
    public int width;
    public int height;
  }
```

## 4. public 클래스 내부의 불변 필드

public 클래스 내부에 불변 필드는 괜찮을까요?

불변 필드라면 클라이언트가 직접 변경할 수도 없기 때문에 언뜻 보기에는 괜찮을 수 있습니다.

하지만 API를 변경하지 않고는 표현 방식을 바꿀 수 없고, 필드를 읽을 때 부수 작업을 수행할 수 없기 때문에

사용하지 않는 것이 좋습니다.

## 5. 정리

이번 포스트는 public 클래스에서 public 필드를 사용할 경우 수반되는 문제점에 대해 알아보았습니다.

공용 API가 아닌 경우에는 package-private이나 private 중첩으로 접근 범위를 줄이거나

공용 API인 경우에는 private 필드를 만들고 public 접근자를 통해 데이터를 주고 받는 방법을 선택합시다.