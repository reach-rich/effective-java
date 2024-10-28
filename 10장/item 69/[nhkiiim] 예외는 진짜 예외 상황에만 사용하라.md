# 예외는 진짜 예외 상황에만 사용하라

- 운이 없다면 언젠가 저런 코드를 마주칠지도 몰라~

```java
try {
  int i = 0;
  while (true) {
    range[i++].climb();
  }
} catch (ArrayIndexOutOfBoundsException e) {
}
```
