### 🔍 비트 필드(bit field)

**✏ #01 예제소스 | 비트 필드 열거 상수 - 구닥다리 기법**

```java
public class Text {
    public static final int STYLE_BOLD			= 1 << 0;
    public static final int STYLE_ITALIC		= 1 << 1;
    public static final int STYLE_UNDERLINE		= 1 << 2;
    public static final int STYLE_STRIKETHROUGH	= 1 << 3;
    
    public void applyStyles(int styles) {...}
}
```

>열거한 값들이 집합으로 사용될 경우 각 상수에 서로 다른 2의 거듭제곱 값을 할당한 정수 열거 패턴을 사용해왔음 (비트 필드)
>
>비트별 연산을 사용해 합집합과 교집합 같은 집합 연산을 효율적으로 수행 가능
>
>단, 정수 열거 상수의 단점을 그대로 지니며 추가 문제도 발생

<br>

---

<br>

### 🤔 비트 필드 열거 상수의 문제점

- 비트 필드 값이 그대로 출력되면 단순한 정수 열거 상수를 출력할 때보다 해석하기 훨씬 어려움
- 비트 필드 하나에 녹아 있는 모든 원소를 순회하기도 까다로움
- 최대 몇 비트가 필요한지를 API 작성 시 미리 예측하여 적절한 타입을 선택해야함 (보통 `int` 또는 `long`)
  - API를 수정하지 않고는 비트 수 (32비트 or 64비트)를 더 늘릴 수 없기 때문

<br>

---

<br>

### 💡 EnumSet 클래스

- `java.util` 패키지의 `EnumSet` 클래스는 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현
- `Set` 인터페이스를 완벽히 구현, 타입 안전, 다른 어떤 `Set` 구현체와 함께 사용 가능
- 내부는 비트 벡터로 구현되어 있음
  - 원소가 64개 이하라면(대부분) `EnumSet` 전체를 `long` 변수 하나로 표현하여 비트 필드에 비견되는 성능을 보여줌
  - `removeAll`과 `retainAll` 같은 대량 작업은 비트를 효율적으로 처리할 수 있는 산술 연산을 써서 구현했음

<br>

**✔ 그럼에도 난해한 작업을 EnumSet이 다 처리해주기 때문에 비트를 직접 다룰 때 흔히 겪는 오류를 방지할 수 있음**

<br>

**✏ #02 예제소스 | EnumSet - 비트 필드를 대처하는 현대적 기법**

```java
public class Text {
    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
    
    // 어떤 Set을 넘겨도 되나, EnumSet이 가장 좋다
    public void applyStyles(Set<Style> styles) {...}
}
```

>`applyStyle` 메서드에 `EnumSet` 인스턴스를 건네는 클라이언트 코드
>
>`EnumSet`은 집합 생성 등 다양한 기능의 정적 팩토리를 제공 

<br>

```java
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```

❓`applyStyles` 메서드가 `EnumSet<Style>` 이 아닌 `Set<Style>`을 받은 이유

>모든 클라이언트가 `EnumSet`을 건네리라 짐작되는 상황이라도 이왕이면 인터페이스로 받는 게 일반적으로 좋은 습관
>
>이렇게 하면 특이한 클라이언트가 다른 `Set` 구현체를 넘기더라도 처리할 수 있음

<br>

---

<br>

### 📌 핵심정리

**열거할 수 있는 타입을 한데 집합 형태로 사용한다고 해도 비트 필드를 사용할 이유는 없다**

**EnumSet 클래스가 비트 필드 수준의 명료함과 성능을 제공하고 열거 타입의 장점까지 선사하기 때문이다**

**EnumSet의 유일한 단점이라면 (자바 9까지는 아직) 불변 EnumSet을 만들 수 없다는 것이다**

**그래도 향후 릴리스에서는 수정되리라 본다**	

**그때까지는 (명확성과 성능이 조금 희생되지만) Collections, unmodifiableSet으로 EnumSet을 감싸 사용할 수 있다**
