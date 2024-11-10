### 📝 예외번역

**: 상위 계층에서는 저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 던지는 것**

**✏ #01 예제소스 | 예외 번역**

```java
try {
    ... // 저수준 추상화를 이용한다.
} catch (LowerLevelException e) {
    // 추상화 수준에 맞게 번역한다.
    throw new HigherLevelException(...);
}
```

```java
public E get(int index) {
    ListIterator<E> i = listIterater(index);
    try {
        return i.next();
    } catch (NoSuchException e) {
        throw new IndexOutOfBoundsException("인덱스: " + index);
    }
}
```

<br>

**✏ #02 예제소스 | 예외 연쇄**

```java
try {
    ... // 저수준 추상화를 이용한다.
} catch (LowerLevelException e) {
    // 저수준 예외를 고수준 예외에 실어 보낸다.
    throw new HigherLevelException(cause);
}
```

>예외 연쇄란 문제의 근본 원인(cause)인 저수준 예외를 고수준 예외에 실어 보내는 방식

<br>

**✏ #03 예제소스 | 예외 연쇄용 생성자**

```java
class HigerLevelEception extends Exception {
    HigherLevelExcpetion (Throwable cause) {
        super(cause);
    }
}
```

>고수준 예외의 생성자는 (예외 연쇄용으로 설계된) 상위 클래스 생성자에 이 '원인'을 건네주어, 최종적으로 Throwable 생성자까지 전달

<br>

**✔ 무턱대고 예외를 전파하는 것보다 예외 번역이 우수한 방법이지만, 남용해서는 안됨**

>가능하다면 저수준 메서드가 반드시 성공하도록하여 아래 계층에서는 예외가 발생하지 않도록 하는 것이 최선

<br>

### 📌 핵심정리

**아래 계층의 예외를 예방하거나 스스로 처리할 수 없고, 그 예외를 상위 계층에 그대로 노출하기 곤란하다면 예외 번역을 사용하라**

**이때 예외 연쇄를 이용하면 상위 계층에는 맥락에 어울리는 고수준 예외를 던지면서 근본 원인도 함께 알려주어 오류를 분석하기에 좋다**

<br>

