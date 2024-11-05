### 📝문자열 연결

- 문자열 연결 연산자로 문자열 n개를 잇는 시간은 n^2 에 비례함

<br>

**✏ #01 예제소스 | 문자열 연결을 잘못 사용한 예 - 느리다**

```java
public String statement() {
    String result = "";
    for (int i = 0; i < numItems(); i++)
        result += lineForItem(i);
    return result;
}
```

>- 품목이 많을 경우 이 메서드는 심각하게 느려질 수 있음

**✔성능을 포기하고 싶지 않다면 `String` 대신 `StringBuilder`를 사용**

<br>

**✏ #02 예제소스 | StringBuilder를 사용하면 문자열 연결 성능이 크게 개선된다**

```java
public String statement2() {
    StringBuilder b = new StringBuilder(numItems() * LINE_WIDTH);
    for (int i = 0; i < numItems(); i++)
        b.append(lineForItem(i));
    return b.toString();
}
```

- `StringBuilder`를 사용했을때 수행시간은 선형으로 증가
- 단, 전체 결과를 담기에 충분한 크기로 초기화(기본값을 사용하더라도 빠름)

<br>

### 📌 핵심정리

**성능에 신경 써야 한다면 많은 문자열을 연결할 때는 문자열 연결 연산자(+)를 피하자**

**대신 StringBuilder의 append 메서드를 사용하라**

**문자 배열을 사용하거나, 문자열을 (연결하지 않고) 하나씩 처리하는 방법도 있다**

<br>
