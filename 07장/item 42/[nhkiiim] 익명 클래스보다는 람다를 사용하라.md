# 익명 클래스보다는 람다를 사용하라


### 1. 익명 클래스
자바에서 함수를 표현할 때 추상 메서드를 하나만 담은 인터페이스를 사용

이런 인터페이스의 인스턴스를 함수 객체라고 하며 특정 함수나 동작을 나타내는데 사용

<br>

1997년 JDK 1.1이 등장하면서 함수 객체를 만드는 주요 수단으로 익명클래스가 쓰임

```java
//익명 클래스의 인스턴스를 함수 객체로 사용 - 낡은 기법!!
Collections.sort(words, new Comparator<String>(){
  public int compare(String s1, String s2){
    return Integer.compare(s1.length(), s2.length());
  }
});
```

<br>

__전략패턴__ 처럼 함수 객체를 사용하는 과거 객체지향 디자인 패턴에는 익명 클래스면 충분!!
- 객체들이 할 수 있는 행위 각각에 대해 전략 클래스를 생성하고, 유사한 행위들을 캡슐화 하는 인터페이스를 정의
- 객체의 행위를 동적으로 바꾸고 싶은 경우 직접 행위를 수정하지 않고 전략을 바꿔주기만 함으로써 행위를 유연하게 확장하는 방법

`하지만` 익명 클래스 방식은 코드가 너무 길기 때문에 자바는 함수형 프로그래밍에 적합하지 않았음!!

#
### 2. 람다식

자바8에 와서 추상 메서드 하나짜리 인터페이스는 특별한 의미를 인정받음!

바로 인터페이스들의 인스턴스를 람다식(lambda expression, 짧게는 람다)을 사용해 만들 수 있게 된 것!

람다는 함수나 익명클래스와 개념은 비슷하지만 코드는 훨씬 간결함

```java
Collections.sort(words, 
  (s1, s2) -> Integer.compare(s1.length(), s2.length()));
```

여기서 람다, 매개변수(s1,s2), 반환값의 타입은 각각 (Comparator<String\>), String, int 지만 코드에서는 언급하지 않아도 됨!

우리 대신 컴파일러가 문맥을 살펴 타입을 추론해줌 (상황에 따라 컴파일러가 타입을 결정하지 못할 경우 프로그래머가 직접 명시해줘야함 -> 하지만 매우 복잡)

__타입을 명시해야 코드가 더 명확해지는 경우 말고는, 람다의 모든 매개변수 타입은 생략해도 된다__

그리고 컴파일러가 "타입을 알 수 없다" 라는 메세지를 줄 때만 해당 타입을 명시하자!


#
### 3. 더 간결한 코드 만들기

__(1) 비교자 생성 메서드 사용__

람다 자리에 비교자 생성 메서드를 사용하면 코드를 더 간결하게 만들 수 있음

```java
Collections.sort(words, comparingInt(String::length);
```

- `::`의 의미 : `::`를 기준으로왼쪽 객체의 오른쪽 메소드를 사용
- comparingInt : https://www.educative.io/answers/what-is-comparatorcomparingint-method-in-java

<br>

__(2) 자바 8 List에 추가 된 sort 메서드 사용__

```java
words.sort(comparingInt(String::length));
```

<br>

__(3) 열거 타입에 적용__ 
```java
public enum Operation {
    PLUS("+", (x, y) -> x+y);
    MINUS("-", (x,y) -> x-y);
    TIMES("*", (x,y) -> x*y);
    DIVIDE("/", (x,y) -> x / y);
    
    private final String symbol;
    private final DoubleBinaryOperator op;
    
    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }
    
    @Override
    public String toString() {
        return symbol;
    }
    
    public double apply(double x, double y) {
        return op.applyAsDouble(x, y);
    }
}
```

#
### 4. 무조건 람다식을 사용해야 할까?

- 메서드나 클래스와 다르게 람다는 이름이 없고 문서화도 못함
- 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 람다를 쓰지 말아야함

- 람다는 한줄일 때 가장 좋고 세줄을 넘어가면 가독성이 나빠짐
- 열거타입 생성자 안의 람다는 열거 타입의 인스턴스 멤버에 접근할 수 없다 (인스턴스가 런타임에 만들어짐)
- 람다는 함수형 인터페이스에서만 쓰이기 때문에 추상클래스의 인스턴스를 만들 때 람다를 사용할 수 없음
- 추상 메서드가 여러 개인 인터페이스의 인스턴스를 만들 때도 람다 대신 익명 클래스를 쓸 수 있음
- 람다는 자기 자신을 참조할 수 없음 -> 자기 자신을 참조하려면 익명클래스 사용
- 직렬화 해야하는 상황이라면 람다와 익명클래스를 사용하지 말고 private 정적 중첩 클래스를 사용해야함






