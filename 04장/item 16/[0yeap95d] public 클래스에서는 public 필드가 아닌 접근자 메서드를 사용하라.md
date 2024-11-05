### ❌ public 필드 사용

**✏ #01 예제소스**

```java
class Point {
    public double x;
    public double y;
}
```

> - 캡슐화의 이점을 제공하지 못한다
> - API를 수정하지 않고는 내부표현을 바꿀 수 없다
> - 불변식을 보장하지 못한다
> - 외부에서 필드에 접근할 때 부수작업을 수행할 수 없다

**필드를 모두 private로 바꾸고 public 접근자(getter)를 추가한다면?**

<br>

---

<br>

### 💡 접근자 메서드 사용

**✏ #02 예제소스 | 접근자와 변경자(mutator) 메서드를 활용해 데이터 캡슐화**

```java
class Point {
    private double x;
    private double y;
    
    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }
    
    public double getX() { return x; }
    public double getY() { return y; }
    
    public void setX(double x) { this.x = x; }
    public void setY(double y) { this.y = y; }
}
```

> - 패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공
> - 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 가짐

<br>

---

<br>

### 🔗 package-private 클래스 혹은 private 중첩 클래스

- 추상 개념만 올바르게 표현한다면, 데이터 필드를 노출한다 해도 문제가 없다
- 클래스 선언 면에서나 이를 사용하는 클라이언트 코드 면에서나 접근자 방식보다 깔끔하다
- 패키지 바깥 코드는 전혀 손대지 않고도 데이터 표현 방식을 바꿀 수 있다.
- private 중첩 클래스의 경우 수정 범위가 더 좁아져 이 클래스를 포함하는 외부 클래스까지로 제한된다

*필드를 직접 노출하는 대표적인 예시로 ```java.awt.package``` 패키지의 ```Point```와 ```Dimension```클래스가 있다*

<br>

---

<br>

### ❓ public 클래스의 필드가 불변이라면

- 직접 노출할 때의 비하면 단점이 조금 줄어들지만 좋지 않다
- 여전히 API를 변경하지 않고는 표현 방식을 바꿀 수 없고, 필드를 읽을 때 부수작업을 수행할 수 없다
- 단, 불변식은 보장할 수 있다

**✏ #03 예제소스 | 불변 필드를 노출한 public 클래스**

```java
public final class Time {
	private static final int HOURS_PER_DAY		= 24;
    private static final int MINUTES_PER_HOUR	= 60;
    
    public final int hour;
    public final int minute;
    
    public Time(int hour, int minute) {
        if (hour < 0 || hour >= HOURS_PER_DAY)
            throw new IllegalArgumentException("시간: " + hour);
        if (minute < 0 || minute >= MINUTES_PER_HOUR)
            throw new IllegalArgumentException("분: " + minute);
        this.hour = hour;
        this.minute = minute;
    }
}
```

<br>

---

<br>

### 📌 핵심정리

**public 클래스는 절대 가변 필드를 직접 노출해서는 안 된다**

**불변 필드라면 노출해도 덜 위험하지만 완전히 안심할 수 는 없다**

**하지만 package-private 클래스나 private 중첩 클래스에서는 종종(불변이든 가변이든) 필드를 노출하는 편이 나을 때도 있다**

<br>
