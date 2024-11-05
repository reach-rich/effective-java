# 톱레벨 클래스는 한 파일에 하나만 담으라

### 1. 톱레벨 클래스
- 탑레벨 클래스(Top Level Classes)는 우리가 일반적으로 지금까지 써온 클래스
- 내부클래스와 이름이 혼동되기 때문에 표현하는 방법 
- 클래스중에 가장 바깥에 즉 가장 최상위에 위치

<br>

- 소스 파일 하나에 톱 레벨 클래스를 여러 개 선언하더라도 자바 컴파일러는 불평하지 않음
- 하지만 아무런 득도 없고 심각한 위험을 감수해야하는 행위
- 클래스를 여러 가지로 정의할 수 있게 되며 어느 것을 사용할 지는 어떤 소스 파일을 먼저 컴파일하냐에 따라 달라짐


#
### 2. 탑클래스가 여러개인 예제
```java
public class Main {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
}
```

__1) Utensil.java__

```java
class Utensil {
    static final String NAME = "pan";
}

class Dessert {
    static final String NAME = "cake";
}
```

<br>

__2) Dessert,java__

```java
class Utensil {
    static final String NAME = "pot";
}

class Dessert {
    static final String NAME = "pie";
}
```

- 메인 클래스와 메인 클래스가 참조하는 두개의 클래스
- javac Main.java Dessert.java 명령어로 컴파일 한다면 운 좋으면 클래스 중복 정의라고 알려줌
- 하지만 컴파일 순서에 따라 동작이 달라지게 됨


#
### 3. 결론은 단순하다 : 톱레벨 클래스를 분리하라!
- 단순한 해결 방법! 톱레벨 클래스를 한 소스 파일에 적지 말라
- 다른 클래스에 딸린 부차적인 클래스라면 정적 멤버 클래스로 만들어라
- 읽기도 좋고, private로 선언하면 접근 범위도 최소로 관리할 수 있다

```java
public class Test {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }

    private static class Utensil {
        static final String NAME = "pan";
    }

    private static class Dessert {
        static final String NAME = "cake";
    }
}
```

