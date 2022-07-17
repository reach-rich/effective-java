### 🔍 인터페이스의 역할

**: 클래스가 어떤 인터페이스를 구현한다는 것은 자신의 인스턴스로 무엇을 할 수 있는지 클라이언트에 전달**

- 인터페이스는 위의 용도로만 사용해야한다

<br>

---

<br>

### ❌ 상수 인터페이스

**: 메서드 없이, 상수를 뜻하는 `static final` 필드로만 가득 찬 인터페이스**

- 이 상수들을 사용하려는 클래스에서는 정규화된 이름을 쓰는 걸 피하고자 그 인터페이스를 구현한다

<br>

**✏ #01 예제소스 | 상수 인터페이스 안티패턴(사용금지)**

```java
public interface PhysicalConstants {
	static final double AVOGADROS_NUMBER	= 6.022_140_857e23;
    static final double BOLTZMANN_CONSTANT	= 1.280_648_52e-23;
    static final double ELECTRON_MASS		= 9.109_383_56e-31;
}
```

> - 상수 인터페이스 안티패턴은 인터페이스를 잘못 사용한 예제
>
> - 내부 구현을 클래스의 API로 노출하는 행위
> - 사용자 혼란, 클라이언트 내부 구현에 해당하는 이 상수들에 종속
> - 다음 릴리스에서 사용하지 않더라도 바이너리 호환성을 위해 상수 인터페이스를 구현하고 있어야 함

<br>

---

<br>

### 💡 상수를 공개하려면

- 특정 클래스나 인터페이스와 강하게 연관된 상수라면 그 클래스나 인터페이스 자체에 추가해야 함
- 모든 숫자 기본 타입의 박싱 클래스인 Integer와 Double에 선언된 MIN_VALUE와 MAX_VALUE가 있다
- 열거 타입으로 나타내기 적합한 상수라면 **열거 타입**으로 만들어 공개
- 열거 타입이 아니라면 인스턴스화할 수 없는 **유틸리티 클래스**에 담아 공개

<br> **✏ #02 예제소스 | 상수 유틸리티 클래스**

```java
package effectivejava.chapter4.item22.constantutilityclass;

public interface PhysicalConstants {
    private PhysicalConstants() { } // 인스턴스화 방지
    
	public static final double AVOGADROS_NUMBER	= 6.022_140_857e23;
    public static final double BOLTZMANN_CONSTANT	= 1.280_648_52e-23;
    public static final double ELECTRON_MASS		= 9.109_383_56e-31;
}
```

> 유틸리티 클래스에 정의된 상수를 클라이언트에서 사용하려면 클래스 이름까지 함께 명시해야함
>
> 유틸리티 클래스의 상수를 빈번히 하숑한다면 정적 임포트하여 클래스 이름은 생략
>
> ```java
> import static effectivejava.chapter4.item22.constantutilityclass.PhysicalConstants.*;
> ```

<br>

---

<br>

### 📌 핵심정리

**인터페이스는 타입을 정의하는 용도로만 사용해야한다
 상수 공개용 수단으로 사용하지 말자!!**

<br>
