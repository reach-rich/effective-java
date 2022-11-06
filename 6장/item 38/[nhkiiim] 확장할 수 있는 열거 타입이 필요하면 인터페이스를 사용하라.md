# 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

### 1. 인터페이스를 활용해 열거 타입 확장하기

열거 타입은 확장할 수 없다 (상속 불가)

열거 타입을 확장해서 사용하려면 인터페이스를 정희하고 열거 타입으로 구현하면 된다

```java
public interface Operation {
    double apply(double x, double y);
}
```

```java
public enum BasicOperation implements Operationn {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };
    
    private final String symbol;
    
    BasicOperation(String symbol) {
        this.symbol = symbol;
    }
    
    @Override
    public String toString() {
        return symbol;
    }
}
```

열거 타입인 BasicOperation은 확장 불가하지만 인터페이스인 Operation은 확장할 수 있다

이 인터페이스를 연산의 타입으로 사용하면 된다

<br>

__예로 앞의 연산 타입에 지수 연산과 나머지 연산을 추가해 보자__

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
    
    @Override
    public String toString() {
        return symbol;
    }
}
```

새로 작성한 연산은 BasicOperation이 아닌 Operation 인터페이스로 작성돼있기만 하면 어디든 쓸 수 있다

1) ExtendedOperation의 확장 연산 출력하기

```java
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(ExtendedOperation.class, x, y); //ExtendedOperation의 class 리터럴을 넘겨 확장된 연산들 알려줌
}

private static <T extends Enum<T> & Operation> void test(Class<T> opEnumType, double x, double y) { //Class 객체가 열거타입이면서 Operation의 하위 타입이어야 한다는 뜻
    for (Operation op : opEnumType.getEnumConstants()) {
        System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
    }
}
```

2) Class 객체 대신 한정적 와일드타드 타입으로 확장 연산 출력하기

```java
public static void main(String[] args) {
    double x = 10;
    double y = 2;
    test(Arrays.asList(ExtendedOperation.values()), x, y);
}

public static void test(Collection<? extends Operation> opSet, double x, double y) {
    for (Operation op : opSet) {
        System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
    }
}
```

방법 1) 보단 덜 복잡하고 유연한 방법

#
### 2. 인터페이스를 활용한 확장 방법의 문제
열거 타입끼리 구현을 상속할 수 없다

아무 상태에도 의존하지 않는 경우 디폴트로 구현하는데 Enum으로 구현하는 경우에는 그럴 수 없다

모든 구현체에 로직을 구현해줘야해서 중복되는 메서드가 발생할 수 있다

<br>

공유하는 기능이 많다면 이러한 문제를 별도의 도우미 클래스나 정적 도우미 메서드르 분리할 수 있다




