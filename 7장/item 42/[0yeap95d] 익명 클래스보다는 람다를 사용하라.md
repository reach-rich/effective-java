### 📖 익명 클래스

- 예전에는 자바에서 함수 타입을 표현할 때 추상 메서드를 하나만 담은 인터페이스(드물게는 추상 클래스)를 사용
- 이런 인스턴스를 함수객체(function object)라고 하여, 특정 함수나 동작을 나타내는 데 사용
- JDK 1.1이 등장하면서 함수 객체를 만드는 주요 수단은 익명 클래스

<br>

**✏ #01 예제소스 | 익명 클래스의 인스턴스를 함수 객체로 사용 - 낡은 기법**

```java
Collections.sort(words, new Comparator<String>() {
   public int compare(String s1, String s2) {
       return Integer.compare(s1.length(), s2.length());
   } 
});
```

>`Comparator` 인터페이스가 정렬을 담당하는 추상 전략을 뜻하며, 문자열을 정렬하는 구체적인 전략을 익명 클래스로 구현
>
>익명 클래스 방식은 코드가 너무 길기 때문에 자바는 함수형 프로그래밍에 적합하지 않음

<br>

---

<br>

### 🔎 람다식

- 자바 8에 와서 추상 메서드 하나짜리 인터페이스는 특별한 의미를 인정받아 특별한 대우를 받음
- 지금은 함수형 인터페이스라 부르는 이 인터페이스들의 인스턴스를 람다식을 사용해 만들 수 있게 됨
- 람다는 함수나 익명 클래스와 개념은 비슷하지만 코드는 훨씬 간결함

<br>

**✏ #02 예제소스 | 람다식을 함수 객체로 사용 - 익명 클래스 대체**

```java
Collections.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length()));
```

>반환값의 타입은 각각 `(Comparator<String>)`, `String`, `int`지만 코드에서는 언급이 없지만 컴파일러가 문맥을 살펴 타입 추론

<br>

**❓ 타입추론은 어떻게 하는가** 

- 상황에 따라 컴파일러가 타입을 결정하지 못하는 경우, 프로그래머가 직접 명시
- 타입 추론 규칙은 매우 복잡하지만 전부 이해하지 못하더라도 상관없이 사용 가능
- 우선 타입을 명시해야 코드가 더 명확할 때만 제외하고는, 람다의 모든 매개변수 타입은 생략
- 이후 컴파일러가 "타입을 알 수 없다"는 오류를 낼 때만 해당 타입을 명시

<br>

**✏ 람다 자리에 비교자 생성 메서드를 사용하면 더 간결하게 가능** 

```java
Collections.sort(words, comparingInt(String::length));
```

<br>

**✏ 자바 8에 List 인터페이스에 추가된 sort 메서드를 이용하면 더욱 짧아짐**

```java
words.sort(comparingInt(String::length));
```

<br>

---

<br>

### 📝 예시_Operation(열거타입)

- 람다를 언어 차원에서 지원하면서 기존에 적합하지 않았던 곳에서도 함수객체를 실용적으로 사용할 수 있게 되었음

<br>

**✏ #03 예제소스 | 상수별 클래스 몸체와 데이터를 사용한 열거 타입**

```java
public enum Operation {
    PLUS("+") {
		public doubld apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public doubld apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public doubld apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public doubld apply(double x, double y) { return x / y; }
    };
    
    private final String symbol;
    
    Operation(String symbol) { this.symbol = symbol; }
    
    @Override public String toString() { return symbol; }
    public abstract double apply(double x, double y);
}
```

>상수별 클래스 몸체를 구현하는 방식보다는 열거 타입에 인스턴스 필드를 두는 편이 낫다고 하였음
>
>람다를 이용하면 열거타입의 인스턴스 필드를 이용하는 방식으로 상수별로 다르게 작동하는 코드를 쉽게 구현 가능

<br>

**✏ #04 예제소스 | 함수 객체(람다)를 인스턴스 필드에 저장해 상수별 동작을 구현**

```java
public enum Operation {
    PLUS ("+", (x, y) -> x + y),
    MINUS ("-", (x, y) -> x - y),
    TIMES ("*", (x, y) -> x * y),
    DIVIDE ("/", (x, y) -> x / y);
    
    private final String symbol;
    private final DoubleBinaryOperator op;
    
    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }
    
    @Override public String toString() { return symbol; }
    
    public double apply(double x, double y) {
        return op.applyAsDouble(x, y);
    }
}
```

>각 열거 타입 상수의 동작을 람다로 구현해 생성자에 넘기고, 생성자는 이 람다를 인스턴스 필드로 저장
>
>그런 다음 `apply` 메서드에서 필드에 저장된 람다를 호출

<br>

💡 ***열거 타입 상수의 동작을 표현한 람다를 DoubleBinaryOperator 인터페이스 변수에 할당***

 *`DoubleBinaryOperator`는 `java.util.function` 패키지가 제공하는 다양한 함수 인터페이스 중 하나로, `Double` 타입 인수 2개를 받아 `Double` 타입 결과를 돌려준다*

<br>

---

<br>

### 🤔 주의사항

**✔코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많은 경우**

- 람다는 이름이 없고 문서화를 하지 못한다
- 람다는 세 줄이 넘어가면 가독성이 심하게 떨어지므로 한 줄일 때 가장 좋고 길어야 세 줄 안에 끝내야 좋다
- 람다가 길거나 읽기 어렵다면 더 간단히 줄여보거나 람다를 쓰지 않는 쪽으로 리팩터링이 필요하다

<br>

**✔상수별 동작을 단 몇 줄로 구현하기 어렵거나, 인스턴스 필드나 메서드를 사용해야만 하는 경우**

- 열거타입 생성자에 넘겨지는 인수들의 타입도 컴파일타임에 추론된다
- 따라서 열거 타입 생성자 안의 람다는 열거 타입의 인스턴스 멤버에 접근할 수 없다 (인스턴스는 런타임에 만들어지기 때문)

<br>

**✔람다로 대체할 수 없는 경우**

- 람다는 함수형 인터페이스에서만 쓰인다
- 예컨대 추상 클래스의 인스턴스를 만들 때 람다를 쓸 수 없으니, 익명 클래스를 써야 한다
- 비슷하게 추상 메서드가 여러 개인 인터페이스의 인스턴스를 만들 때도 익명 클래스를 쓸 수 있다

<br>

**✔함수 객체가 자신을 참조해야 하는 경우**

- 람다는 자기 자신을 참조할 수 없다
- 람다에서의 `this` 키워드는 바깥 인스턴스를, 익명 클래스에서의 `this`는 익명 클래스의 인스턴스 자신을 가리킨다
- 그래서 함수 객체가 자신을 참조해야 한다면 반드시 익명 클래스를 써야한다

<br>

**✔ 직렬화해야만 하는 함수 객체가 있는 경우**

- 람다도 익명 클래스처럼 직렬화 형태가 구현별로 다를 수 있다
- 따라서 람다를 직렬화하는 일은 극히 삼가야 한다
- 직렬화해야만 하는 함수 객체가 있다면(가령 `Comparator`) `private` 정적 중첩 클래스의 인스턴스를 사용하자

<br>

---

<br>

### 📌 핵심정리

**자바가 8로 판올림되면서 작은 함수 객체를 구현하는 데 적합한 람다가 도입되었다**

**익명 클래스는 (함수형 인터페이스가 아닌) 타입의 인스턴스를 만들 때만 사용하라**

**람다는 작은 함수 객체를 아주 쉽게 표현할 수 있어 (이전 자바에서는 실용적이지 않던) 함수형 프로그래밍의 지평을 열었다**

<br>

