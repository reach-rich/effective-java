### ❓ 불변클래스란

**: 인스턴스의 내부 값을 수정할 수 없는 클래스**

- `String`, 기본 타입의 박싱된 클래스들, `BigInteger`, `BigDecimal`

<br>

---

<br>

### ⚙ 불변클래스 규칙

- 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다

- 클래스를 확장할 수 없도록 한다

- 모든 필드를 ```final```로 선언한다 

- 인스턴스를 동기화 없이 다른 스레드로 건네도 문제없이 동작하게끔 보장한다

- 모든 필드를 `private`으로 선언한다

- 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다



**✏ #01 예제소스 | 불변 복소수 클래스**

```java
public final class Complex {
    private final double re;
    private final double im;
    
    public double realPart()		{ return re; }
    public double imageinaryPart()	{ return im; }
    
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
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp, 
                            im * c.re - re * c.im) / tmp);
    }
    
    @Override public boolean equals(Object o) {
        if (o == this) return true;
        if (!(o instanceof Complex)) return false;
        Complex c = (Complex) o;
        
        return Double.compare(c.re, re) == 0 && Double.compare(c.im, im) == 0;
    }
    
    @Override public int hashCode() {
		return 31 * Double.hashCode(re) + Double.hashCOde(im);
    }
    
    @Override public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
```

> 메서드들이 인스턴스 자신은 수정하지 않고 새로운 ```Complex``` 인스턴스를 만들어 반환
>
> **함수형 프로그래밍**
>
> - 피연산자에 함수를 적용해 그 결과를 반환하지만, 피연산자 자체는 그대로인 프로그래밍 패턴이다
>
> - 코드에서 불변이 되는 영역의 비율이 높아지는 장점
>
> 절차적 혹은 명령형 프로그래밍에서는 메서드에서 피연산자인 자신을 수정해 자신의 상태가 변함
>
> 메서드 이름에 동사(add) 대신 전치사(plus) 사용함으로서 객체의 값을 변경하지 않는다는 사실을 강조 ~~이런 깊은 뜻이...~~

<br>

---

<br>

### 🗝 불변클래스의 장점

**불변클래스 동기화 & 재활용**

- 불변 객체는 근본적으로 스레드 안전하여 따로 동기화할 필요 없다

- 여러 스레드 동시 사용해도 절대 훼손되지 않는다

- 불변 객체는 안심하고 공유할 수 있다

- 불변 클래스라면 한번 만든 인스턴스를 최대한 재활용 

- 자주 쓰이는 값들을 상수(public static final)로 제공

  > ```java
  > public static final Complex ZERO 	= new Complex(0, 0);
  > public static final Complex ONE 	= new Complex(1, 0);
  > public static final Complex I 		= new Complex(0, 1);
  > ```

<br>

**불변클래스 인스턴스 캐싱**

- 자주 사용되는 인스턴스를 캐싱하여 같은 인스턴스를 중복 생성하지 않게 해주는 정적 팩터리 제공 가능

- 박싱된 기본 타입 클래스 전부, `BigInteger`

- 정적 팩토리 사용 시 여러 클라이언트가 인스턴스를 공유하여 메모리 사용량과 가비지 컬렉션 비용이 감소

<br>

**불변클래스 복사 & 공유**

- 아무리 복사해봐야 원본과 똑같으니  방어적 복사도 필요 없다

- 불변 클래스는 ```clone``` 메서드나 복사 생성자를 제공하지 않는 게 좋다

  *String 클래스의 복사 생성자는 이 사실을 잘 이해하지 못한 자바 초창기에 만들어진것으로 되도록 사용 X*

<br>

**그 외**

- 불변 객체는 자유롭게 공유할 수 있음은 물론, 불변 객체끼리는 내부 데이터를 공유할 수 있다

  > **BigInteger 클래스**
  >
  > 내부에서 값의 부호(sign)와 크기(magnitude)를 따로 표현
  >
  > 부호에는 int변수를, 크기(절대값)에는 int배열을 사용

  > **negate 메서드**
  >
  > 크기는 같고 부호만 반대인 새로운 `BigInteger`를 생성
  >
  > 배열은 비록 가변이지만 복사하지 않고 원본 인스턴스와 공유 가능
  >
  > 새로 만든 `BigInteger`인스턴스도 원본 인스턴스가 가리키는 내부 배열을 그대로 가리킨다

- 객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 이점이 많다

  > 구조가 복잡하더라도 불변식 유지하기 훨씬 수월
  >
  > 불변객체는 맵의 키와 집합(Set)의 원소로 쓰기에 안성맞춤
  >
  > 맵이나 집합은 안에 담긴 값이 바뀌면 불변식이 허물어지는데, 불변 객체를 사용하면 걱정 없음

- 불변 객체는 그 자체로 실패 원자성을 제공한다

<br>

---

<br>

### 🔒 불변 클래스의 단점

값이 다르면 반드시 독립된 객체로 만들어야한다

**: 가짓수가 많다면 모두 만드는 데 큰 비용이 필요**

```java
BigInteger moby = ...;
moby = moby.flipBit(0);
```

> 백만 비트짜리 ```BigInteger```에서 비트 하나를 변경하는 경우 ```flipBit``` 메서드는 새로운 ```BigInteger``` 인스턴스를 생성
>
> ```BigInteger```의 크기에 비례해 시간과 공간 필요
>
> ```BitSet``` 클래스를 이용하면 비트 하나만 상수 시간에 변경하는 메서드 제공
>
> ```java
> BitSet moby = ...;
> moby = moby.flip(0);
> ```

<br>

**원하는 객체를 완성하기까지 중간 단계가 많고, 중간에 생성된 객체가 버려지는 경우**

1. ```package-private```의 가변 동반 클래스 

   > 다단계 연산들을 예측하여 기본 기능으로 제공
   >
   > ```BigInteger```는 모듈러 지수 같은 다단계 연산 속도를 높여주는 가변 동반 클래스를 ```package-private```으로 두고 있다

2. ```public```의 가변 동반 클래스

   > ```String```의 가변 동반 클래스는 ```StringBuilder```(~~자매품~~ 구버전 ```StringBuffer```) 

<br>

---

<br>

### 🛠 불변 클래스 설계 방법

**모든 생성자를 private 혹은 package-private로 만들고 public 정적 팩터리를 제공**

**✏ #02 예제소스**

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
}
```

<br>

**BigInteger & BigDecimal**

> 신뢰할 수 없는 클라이언트로부터 BigInteger 또는 BigDecimal의 인스턴스를 인수로 받는다면 주의
>
> 인수로 받는 객체가 '진짜'인지 확인해야 한다
>
> ```java
> public static BigInteger safeInstance(BigInteger val) {
> 	return val.getClass() == BigInteger.class ? val : new BigInteger(val.toByteArray());
> }
> ```

<br>

---

<br>

### 📌 핵심정리

**클래스는 꼭 필요한 경우가 아니라면 불변이어야 한다**

**하지만 모든 클래스를 불변으로 만들 수는 없으므로 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이자**

**꼭 변경이 필요한 필드를 뺀 나머지 모두를 final로 선언**

**다른 합당한 이유가 없다면 모든 필드는 private final이어야 한다**

**생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야 한다**

<br>

✔ java.util.concurrent 패키지의 CountDownLatch 클래스
