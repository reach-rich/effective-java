### ⚔ 열거 타입 vs 타입 안전 열거 패턴

**: 열거 타입은 거의 모든 상황에서 타입 안전 열거 패턴(typesafe enum pattern)보다 우수하다!**

<br>

**💡 예외의 경우**

- 타입 안전 열거 패턴은 확장할 수 있으나 열거 타입은 그럴 수 없다
- 타입 안전 열거 패턴은 열거한 값들을 그대로 가져온 다음 값을 더 추가하여 다른 목적으로 사용 가능 (열거 타입은 불가능)

<br>

**✔ 열거 타입을 확장하는건 좋지 않은 생각이다**

- 확장한 타입의 원소는 기반 타입의 원소로 취급하지만 그 반대는 성립하지 않는다면 이상하다 
- 기반 타입과 확장된 타입들의 원소 모두를 순회할 방법도 없음
- 확장성을 높이려면 고려할 요소가 늘어나 설계와 구현이 더 복잡해진다

<br>

**📝 확장할 수 있는 열거 타입이 어울리는 경우 - 연산코드**

- 연산 코드의 각 원소는 특정 기계가 수행하는 연산
- API가 제공하는 기본 연산 외에 사용자 확장 연산을 추가할 수 있도록 열어줘야 하는 경우 발생

<br>

**🔑 열거 타입으로 이 효과를 내는 방법**

- 연산 코드용 인터페이스를 정의하고 열거 타입이 이 인터페이스를 구현하게 하면 된다
- 이때 열거 타입이 그 인터페이스의 표준 구현체 역할을 한다

<br>

---

<br>

### 📙 인터페이스 이용

**✏ #01 예제소스 | 인터페이스를 이용해 확장 가능 열거 타입을 흉내**

```java
public interface Operation {
    double apply(double x, double y);
}

public enum BasicOperation implements Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        ...
    }
    ...
}

private final String symbol;

BasicOperation(String symbol) {
    this.symbol = symbol;
}

@Override public String toString() {
    return symbol;
}
```

>열거 타입인 `BasicOperation`은 확장할 수 없지만 인터페이스인 `Operation`은 확장 가능하므로, 
>이 인터페이스를 연산의 타입으로 사용하면 된다
>
>`Operation`을 구현한 또 다른 열거 타입을 정의해 기본 타입인 `BasicOperation`을 대체할 수 있다

<br>

**✏ #02 예제소스 | 확장 가능 열거 타입**

```java
public enum ExtendedOperation implements Operation {
	EXP("^") {
		public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
		public double apply(double x, double y) {
            return x % y;
        }
    };
	
    private final String symbol;
    
    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }
    
    @Override public String toString() {
        return symbol;
    }
}
```

>새로 작성한 연산은 기존 연산을 쓰던 곳이면 어디든 쓸 수 있음
>
>`Operation`인터페이스를 사용하도록 작성되어 있기만하면 된다
>
>`apply`가 인터페이스에 선언되어 있으니 열거 타입에 따로 추상 메서드를 선언하지 않아도 된다



<br>

---

<br>

### 📗 한정적 와일드카드 사용

**: `Class` 객체 대신 한정적 와일드카드 타입인 `Collection<? extends Operation>`을 넘기는 방법**

<br>

**✏ #03 예제소스**

```java
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(Arrays.asList(ExtendedOperation.values()), x, y);
}

private static void test(Collection<? extends Operation> opSet, double x, double y) {
    for (Operation op : opSet) {
        System.out.println("%f %s %f = %f%n", x, op, y, op.apply(x, y));
    }
}
```

>여러 구현 타입의 연산을 조합해 호출할 수 있게 되었음
>
>반면, 특정 연산에서는 `EnumSet`과 `EnumMap`을 사용하지 못함

<br>

---

<br>

### ❌ 문제점

**🤔 열거 타입끼리 구현을 상속할 수 없음**

- 아무 상태에도 의존하지 않는경우에는 디폴트 구현을 이용해 인터페이스에 추가하는 방법이 있음

- `Operation` 예는 연산 기호를 저장하고 찾는 로직이 `BasicOperation`과 `ExtendedOperation` 모두에 들어가야만 한다

  중복량이 적으니 문제되진 않지만, 공유하는 기능이 많다면 그 부분을 별도의 도우미 클래스나 정적 도우미 메서드로 분리하는 방식으로 코드 중복을 없앨 수 있다

<br>

*✔`java.nio.file.LinkOption` 열거 타입은 `CopyOption`과 `OpenOption` 인터페이스를 구현했다*

<br>

---

<br>

### 📌 핵심정리

**열거 타입 자체는 확장할 수 없지만, 인터페이스와 그 인터페이스를 구현하는 기본 열거 타입을 함께 사용해 같은 효과를 낼 수 있다**

**이렇게 하면 클라이언트는 이 인터페이스를 구현해 자신만의 열거 타입(혹은 다른 타입)을 만들 수 있다**

**그리고 API가 (기본 열거 타입을 직접 명시하지 않고) 인터페이스 기반으로 작성되었다면 기본 열거 타입의 인스턴스가 쓰이는 모든 곳을 새로 확장한 열거 타입의 인스턴스로 대체해 사용할 수 있다**

<br>
