### 🔎 @Override 애너테이션

**: 상위 타입의 메서드를 재정의**

- 일괄되게 사용해야 함으로써 여러 가지 악명 높은 버그들을 예방할 수 있음

<br>

**✏ #01 예제소스 | 영어 알파벳 2개로 구성된 문자열을 표현하는 클래스 - 버그발생!**

```java
public class Bigram {
    private final char first;
    private final char second;
    
    public Bigram(char first, char second) {
        this.first = first;
        this.second = second;
    }
    
    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }
    
    public int hashCode() {
        return 31 * first + second;
    }
    
    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++)
            for (char ch = 'a'; ch <= 'z'; ch++)
                s.add(new Bigram(ch, ch));
        System.out.println(s.size());
    }
}
```

>`Set`은 중복을 허용하지 않으므로 26이 출력되어야 할 것 같지만, 260이 출력됨
>
>`equals`가 재정의`overriding` 이 아닌 다중정의`overloading` 으로 선언되어 있음
>
>재정의를 하기 위해서는 매개변수 타입을 `Object`로 해야함

<br>

**✏ #02 예제소스 | @Override 애너테이션 사용 시**

```java
@Override public boolean equals(Bigram b) {
    return b.first == first && b.second = second;
}
```

>`@Override` 애너테이션을 달고 다시 컴파일하면 컴파일 오류가 발생

<br>

**✏ #03 예제소스 | 올바른 소스**

```java
@Override public boolean equals(Object o) {
    if (!(o instanceof Bigram))
        return false;
    Bigram b = (Bigram) o;
    return b.first == first && b.second = second;
}
```

<br>

**✔ 상위 클래스의 메서드를 재정의하려는 모든 메서드에 @Override 애너테이션을 달자**

<br>

---

<br>

### 👌 예외의 경우

- 구체 클래스인데 아직 구현하지 않은 추상 메서드가 남아 있다면 컴파일러가 그 사실을 바로 알려주기 때문에 메서드를 재정의할 때는 굳이 `@Override`를 달지 않아도 된다

<br>

---

<br>

### 📚 그 외

- `@Override`는 클래스뿐 아니라 인터페이스의 메서드를 재정의할 때도 사용할 수 있음

- 디폴트 메서드를 지원하기 시작하면서, 인터페이스 메서드를 구현한 메서드에도 `@Override`를 다는 습관을 들이면 시그니처가 올바른지 재차 확신 할 수 있음

- 구현하려는 인터페이스에 디폴트 메서드가 없을을 안다면 이를 구현한 메서드에서는 `@Override`를 생략해 코드를 조금 더 깔끔히 유지해도 좋음

- 추상 클래스나 인터페이스에서는 상위 클래스나 상위 인터페이스의 메서드를 재정의하는 모든 메서드에 `@Override`를 다는 것이 좋음

  *`Set` 인터페이스는 `Collection` 인터페이스를 확장했지만 새로 추가한 메서드는 없음*

  *따라서 모든 메서드 선언에 `@Override`를 달아 실수로 추가한 메서드가 없음을 보장했다*

<br>

---

<br>

### 📌 핵심정리

**재정의한 모든 메서드에 @Override 애너테이션을 의식적으로 달면 여러분이 실수했을 때 컴파일러가 바로 알려줄 것이다**

**예외는 한 가지 뿐이다**

**구체 클래스에서 상위 클래스의 추상 메서드를 재정의한 경우엔 이 애너테이션을 달지 않아도 된다(단다고 해서 해로울 것도 없다)**
