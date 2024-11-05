## 1. 들어가기

Java는 소스 파일 하나 당 톱레벨 클래스를 하나만 사용하는 것이 관례입니다.

하지만, 자바 컴파일러는 두 개 이상이더라도 오류를 뱉지 않습니다.

그럼 소스 파일 하나 당 두 개 이상의 톱레벨 클래스를 사용해도 무방하지 않을까요?

이와 관련해 예시를 살펴보겠습니다.

## 2. 문제점

Utensil 클래스와 Dessert 클래스가 Utensil.java라는 한 파일에 정의되어 있다고 가정하겠습니다.

✏️ Utensil.java

```java
  class Utensil {
    static final String NAME = "pan";
  }

  class Dessert {
    static final String NAME = "cake";
  }
```

---

✏️ Dessert.java

```java
  class Utensil {
    static final String NAME = "pot";
  }

  class Dessert {
    static final String NAME = "pie";
  }
```

---

✏️ Main.java

```java
  public class Main {
    public static void main(String[] args) {
      System.out.println(Utensil.NAME + Dessert.NAME);
    }
  }
```

---

🤔 ```javac Main.java Dessert.java``` 명령으로 컴파일한다면 어떻게 될까요?

1. 가장 먼저 Main.java를 컴파일합니다.

2. 그 안에서 Utensil 참조를 만나면 Utensil.java 파일을 살펴 Utensil과 Dessert를 찾아냅니다.

3. ```Utensil.NAME + Dessert.NAME```(pancake)을 출력합니다.

🤔 그럼 ```javac Dessert.java Main.java``` 명령으로 컴파일한다면 어떻게 될까요?

1. 가장 먼저 Dessert.java를 컴파일합니다.

2. 그 안에서 Utensil과 Dessert를 찾아냅니다.

3. ```Utensil.NAME + Dessert.NAME```(potpie)을 출력합니다.

이처럼 컴파일러에 어느 소스 파일을 먼저 건네느냐에 따라 동작이 달라지므로 반드시 수정해야 합니다.

그럼 어떻게 해결할 수 있을까요?

## 3. 해결 방법

해결 방법은 단순히 톱레벨 클래스들을 서로 다른 소스 파일로 분리하면 됩니다.

다음과 같이 말이죠.

✏️ Utensil.java

```java
  class Utensil {
    static final String NAME = "pan";
  }
```

✏️ Dessert.java

```java
  class Dessert {
    static final String NAME = "cake";
  }
```

만약, 여러 톱레벨 클래스를 한 파일에 담고 싶다면 정적 멤버 클래스를 사용해봅시다.

정적 멤버 클래스를 사용하게 되면 읽기 좋고 접근 관리자를 통한 접근 범위도 관리할 수 있습니다.

```java
  public class Main {
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

## 4. 정리

이번 포스트는 한 소스 파일에 여러 톱레벨 클래스를 작성하는 경우의 문제점과 해결 방법을 알아보았습니다.

이왕이면 소스 파일 하나에 반드시 하나의 톱레벨 클래스를 작성하고

굳이 여러 톱레벨 클래스를 한 소스 파일에 담고 싶다면 정적 멤버 클래스를 고려해봅시다.