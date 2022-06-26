Effective Java의 열일곱 번째 아이템 "변경 가능성을 최소화하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 1. 불변 클래스

불변 클래스란 **인스턴스의 내부 값을 수정할 수 없는 클래스**를 말한다. 불변 인스턴스에 간직된 정보는 고정되어 객체가 파괴되는 순간까지 절대 달라지지 않는다. String, 기본 타입의 박싱된 클래스들, BigInteger, BigDecimal이 여기에 속한다.

불변 클래스는 가변 클래스보다 설계하고 구현하고 사용하기 쉬우며, 오류가 생길 여지도 적고 훨씬 안전하다.

<br>

클래스를 불변으로 만들려면 다음 **다섯 가지 규칙**을 따라야 한다.

- 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다.
- 클래스를 확장할 수 없도록 한다.
  - 하위 클래스에서 부주의하게 혹은 나쁜 의도로 객체의 상태를 변하게 만드는 사태를 막아준다.
- 모든 필드를 final로 선언한다.
  - 시스템이 강제하는 수단을 이용해 설계자의 의도를 명확히 드러내는 방법이다.
  - 새로 생성된 인스턴스를 동기화 없이 다른 스레드로 건네도 문제없이 동작하게끔 보장한다.
- 모든 필드를 private으로 선언한다.
  - 필드가 참조하는 가변 객체를 클라이언트에서 직접 접근해 수정하는 일을 막아준다. 
- 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.
  - 클래스에 가변 객체를 참조하는 필드가 하나라도 있다면 클라이언트에서 그 객체의 참조를 얻을 수 없도록 해야 한다.
  - 생성자, 접근자, readObject 메서드 모두에서 방어적 복사를 수행하라.

<br>

## 2. 예제

다음은 복수수를 표현한 불변 클래스의 예제이다.

```java
public final class Complex {
  
  private final double re;
  private final double im;
  
  public Complex(double re, double im) {
    this.re = re;
    this.im = im;
  }
  
  public double readPart() { return re; }
  public double imaginaryPart() { return im; }
  
  public Complex plus(Complex c) {
    return new Complex(re + c.re, im + c.im);
  }
  
  public Complex minus(Complex c) {
    return new Complex(re - c.re, im - c.im);
  }
  
  public Complex times(Complex c) {
    return new Complex(re * c.re - im * c.im,
                      re * c.im + im * c.re);
  }
  
  public Complex dividedBy(Complex c) {
    return new Complex((re * c.re + im * c.im) / tmp,
                      (im * c.re - re * c.im) / tmp);
  }
  
  @Override
  public boolean equals(Object o) {
    if (o == this) return true;
    if (!(o instanceof Complex)) return false;
    Complex c = (Complex) o;
    
    return Double.compare(c.re, re) == 0
      && Double.compare(c.im, im) == 0;
  }
  
  @Override
  public int hashCode() {
    return 31 * Double.hashCode(re) + Double.hashCode(im);
  }
  
  @Override
  public String toString() {
    return "(" + re + " + " im + "i)";
  }
}
```

실수부와 허수부 값을 반환하는 접근자 메서드와 사칙연산 메서드를 정의했다. 이 사칙연산 메서드들이 인스턴스 자신은 수정하지 않고 새로운 Complex 인스턴스를 만들어 반환하는 모습에 주목하자. 또한 메서드 이름으로 (add 같은) 동사 대신 (plus) 같은 전치사를 사용한 점에도 주목하자.

<br>

## 3. 장단점

불변 객체는 다음과 같은 **장점**이 있다.

- **불변 객체는 단순하다.**
  - 불변 객체는 생성된 시점의 상태를 파괴할 때까지 그대로 간직한다. 모든 생성자가 클래스 불변식을 보장한다면 그 클래스르 사용하는 프로그래머가 다른 노력을 들이지 않더라도 영원히 불변으로 남는다.
- **불변 객체는 근본적으로 thread-safe하여 따로 동기화할 필요 없다.**
  - 여러 스레드가 동시에 사용해도 절대 훼손되지 않는다.
  - 불변 객체에 대해서는 그 어떤 스레드도 다른 스레드에 영향을 줄 수 없으니 불변 객체는 안심하고 공유할 수 있다.
  - 따라서 불변 클래스라면 한번 만든 인스턴스를 최대한 재활용하기를 권한다.
    - ex) 상수(public static final), 캐싱
- **객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 이점이 많다.**
  - 값이 바뀌지 않는 구성요소들로 이뤄진 객체라면 그 구조가 아무리 복잡하더라도 불변식을 유지하기 훨씬 수월하기 때문이다.
    - ex) Map의 key, Set의 element
- **불변 객체는 그 자체로 실패 원자성을 제공한다.**
  - 상태가 절대 변하지 않으니 잠깐이라도 불일치 상태에 빠질 가능성이 없다.

<br>

불변 객체에도 **단점**이 있다.

- **값이 다르면 반드시 독립된 객체로 만들어야 한다.**
  - 값의 가짓수가 많다면 이를 모두 만드는 데 큰 비용을 치러야 한다.
  - 원하는 객체를 완성하기까지의 단계가 많고, 그 중간 단계에서 만들어진 객체들이 모두 버려진다면 성능 문제가 더 불거진다.

<br>

위와 같은 문제에 **대처하는 방법**은 두 가지다.

- **흔히 쓰일 다단계 연산들을 예측하여 기본 기능으로 제공하는 방법**
  - 이러한 다단계 연산을 기본으로 제공한다면 더 이상 각 단계마다 객체를 생성하지 않아도 된다.
- **가변 동반 클래스**
  - 클라이언트들이 원하는 복잡한 연산들을 정확히 예측할 수 있다면 package-private의 가변 동반 클래스만으로 충분하다. 그렇지 않다면 이 클래스를 public으로 제공하는 게 최선이다.
    - ex) String - StringBuilder

<br>

## 4. 설계 방법

본문을 시작하며 5가지 규칙을 통해 불변 클래스를 만드는 기본적인 방법을 알아보았다. 추가적으로 **또 다른 설계 방법 두 가지**를 알아본다.

- **모든 생성자를 private 혹은 packge-private으로 만들고 public 정적 팩터리를 제공하여 자신을 상속하지 못하게 한다.**
  - 바깥에서 볼 수 없는 package-private 구현 클래스를 원하는 만큼 만들어 활용할 수 있으니 final 클래스로 만드는 것보다 훨씬 유연하다.
  - 다수의 구현 클래스를 활용한 유연성을 제공한다. 
  - 다음 릴리스에서 객체 캐싱 기능을 추가해 성능을 끌어올릴 수도 있다.

```java
public class Complex {
  
  private final double re;
  private final double im;
  
  private Complex(double re, double im) {
    this.re = re;
    this.im = im;
  }
  
  public static Complex valueOf(double re, double im) {
    return new Complex(re, im);
  }
  
  ...
}
```

- **어떤 메서드도 객체의 상태 중 외부에 비치는 값을 변경할 수 없도록 한다.**
  - 어떤 불변 클래스는 계산 비용이 큰 값을 나중에 (처음 쓰일 때) 계산하여 final이 아닌 필드에 캐시해놓기도 한다. 똑같은 값을 다시 요청하면 캐시해둔 값을 반환하여 계산 비용을 절감하는 것이다.
  

<br>

## 5. 핵심 정리

- 클래스는 꼭 필요한 경우가 아니라면 불변이어야 한다.
- 단순한 값 객체는 항상 불변으로 만들자.
- 무거운 값 객체도 불변으로 만들 수 있는지 고심해야 한다. 성능 때문에 어쩔 수 없다면 불변 클래스와 쌍을 이루는 가변 동반 클래스를 public 클래스로 제공하도록 하자.
- 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이자.
- 다른 합당한 이유가 없다면 모든 필드는 private final이어야 한다.
- 생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야 한다.

<br>

## 6. Related Posts

- readObject 메서드 (Item 88)
- [정적 팩터리 (Item 1)](https://heung27.github.io/posts/effective-java-item-1-%EC%83%9D%EC%84%B1%EC%9E%90-%EB%8C%80%EC%8B%A0-%EC%A0%95%EC%A0%81-%ED%8C%A9%ED%84%B0%EB%A6%AC-%EB%A9%94%EC%84%9C%EB%93%9C%EB%A5%BC-%EA%B3%A0%EB%A0%A4%ED%95%98%EB%9D%BC/)
- 방어적 복사 (Item 50)
- 실패 원자성 (Item 76)
