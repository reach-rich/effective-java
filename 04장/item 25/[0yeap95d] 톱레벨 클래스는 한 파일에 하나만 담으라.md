### 🔒 여러 개의 톱 레벨 클래스

- 소스 파일 하나에 톱레벨 클래스를 여러 개 선언하는 것은 문제없음

- 그러나 아무런 득도 없고 심각한 위험이 존재

<br>

**✏ #01 예제소스 | Utensil.java**

```java
class Utensil { static final String NAME = "pan"; }
class Dessert { static final String NAME = "cake"; }
```

**✏ #02 예제소스 | Dessert.java**

```java
class Utensil { static final String NAME = "pot"; }
class Dessert { static final String NAME = "pie"; }
```

>```java
>public class Main {
>public static void main(String[] args) {
>   System.out.println(Utensil.NAME + Dessert.NAME);
>}
>} 
>```
>
>**⚙ `javac Main.java Dessert.java` 명령으로 컴파일** 
>
>컴파일 오류가 나고 `Utensil`과 `Dessert`클래스가 중복 정의되었음을 알려준다
>
>1. `Main.java` 컴파일 
>2. `Utensil.java` 파일에서 `Utensil`과 `Dessert` 찾음
>3. `Dessert.java`처리 시 같은 클래스 정의 발견
>
><br>
>
>**⚙`javac Main.java` 또는 `javac Main.java Utensil.java` 명령으로 컴파일**
>
>`Dessert.java`파일을 작성하기 이전과 같이 `pancake` 출력
>
><br>
>
>**⚙`javac Dessert.java Main.java` 명령으로 컴파일**
>
>`potpie` 출력

<br>

**✔컴파일러에 어느 소스 파일을 먼저 건네느냐에 따라 동작이 달라지므로 반드시 해결해야 함**

<br>

---

<br>

### 🗝 해결 방법

**1. 단순히 톱레벨 클래스들을 서로 다른 소스 파일로 분리하는 방법**

**2. 정적 멤버 클래스 사용하는 방법** (굳이 사용하고 싶다면)

> **✏ #03 예제소스 | 톱레벨 클래스를 정적 멤버 클래스로**
>
> ```java
> public class Test {
>  public static void main(String[] args) {
>      System.out.println(Utensil.NAME + Dessert.NAME);
>  }
> 	private static class Utensil { static final String NAME = "pan"; }
> 	private static class Dessert { static final String NAME = "cake"; }
> }
> ```

<br>

---

<br>

### 📌 핵심 정리

**소스 파일 하나에는 반드시 톱레벨 클래스(혹은 톱레벨 인터페이스)를 하나만 담자!**

<br>
