# 문자열 연결은 느리니 주의하라.

<br>

## 문자열 연결 연산자(+)
여러 문자열을 하나로 합쳐주는 편리한 수단이지만, 본격적으로 사용하기 시작하면 성능이 저하될 수 있다.
문자열은 불편이라서 두 문자열을 연결하는 경우 양쪽의 내용을 모두 복사해야 하기 때문이다.
문자열 연결 연산자로 문자열 n개를 잇는 시간은 n^2이 비례한다. 

<br>

## StringBuilder
성능을 포기하고 싶지 않다면 StringBuilder를 사용하자.
```java
public String statement2() {
  StringBuilder b = new StringBuilder(numItems() * LINE_WIDTH);
  for (int i = 0; i < numItems(); i++) {
    b.append(lineForItem(i));
  }
  return b.toString();
}
```
