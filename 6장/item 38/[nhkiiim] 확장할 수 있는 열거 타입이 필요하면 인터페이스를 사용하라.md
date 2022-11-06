# 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

### 1. 인터페이스를 활용해 열거 타입 확장하기

열거 타입은 확장할 수 없다

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


__예로 앞의 연산 타입에 지수 연산과 나머지 연산을 추가해 보자__
