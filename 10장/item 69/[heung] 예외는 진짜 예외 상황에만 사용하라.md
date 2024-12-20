## 예외는 진짜 예외 상황에서 사용하라. 

<br>

```java
try {
  int i = 0;
  while (true)
    range[i++].climb();
} catch (ArrayIndexOutOfBoundsException e) {
}
```

위 코드는 무한루프를 돌다가 ArrayIndexOutOfBoundsException이 발생하면 끝을 낸다.
위 코드를 표준적인 관용구대로 작성하면 아래와 같다.

<br>

```java
for (Mountain m : range)
  m.climb();
```

예외를 써서 루프를 종료한 이유는 잘못된 추론을 근거로 성능을 높여보려 한 것이다. 
JVM은 배열에 접근할 때마다 경계를 넘지 않는지 검사하는데, 일반적인 반복문도 배열 경계에 도달하면 종료한다. 
따라서 이 검사를 반복문에도 명시하면 같은 일이 중복되므로 하나를 생략한 것이다. 

<br>

하지만 세 가지 면에서 잘못된 추론이다. 
* 예외는 예외 상황에 쓸 용도로 설계되었으므로 JVM 구현자 입장에서는 명확한 검사만큼 빠르게 만들어야 할 동기가 약하다.
* 코드를 try-catch 블록 안에 넣으면 JVM이 적용할 수 있는 최적화가 제한된다.
* 배열을 순화하는 표준 관용구는 앞서 걱정한 중복 검사를 수행하지 않는다. JVM이 알아서 최적화해 없애준다.

실상은 예외를 사용한 쪽이 표준 관용구보다 훨씬 느리다. 
심지어 반복문 안에 버그가 있다면 제대로 동작하지 않을 수도 있다.

<br>

* **예외는 오직 예외 상황에서만 써야 한다. 절대로 일상적은 제어 흐름용으로 쓰여선 안 된다.**
* **잘 설계된 API라면 클라이언트가 정상적인 제어 흐름에서 예외를 사용할 일이 없게 해야 한다.**

<br>
