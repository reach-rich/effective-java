# int 상수 대신 열거 타입을 사용하라

### 1. 정수 열거 패턴

- 자바에서 열거 타입을 지원하기 전에는 정수 상수들을 선언해 사용하곤 함

```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;

public static final int ORANGE_NAVEL = 0
public static final int ORANGE_TEMPLE = 1;
```

- __정수 열거 패턴에는 단점이 많음__

  - 타입 안전을 보장할 방법이 없으며 표현력이 좋지 않음
  
  - 평범한 상수를 나열한거라 컴파일하면 값이 클라이언트 파일에 그대로 새겨짐 (상수의 값이 바뀌면 다시 컴파일 해야함)
  - 단순 숫자 값을 출력해 문자열으로 출력하기 어려움
  - 상수 묶음의 상수들이 총 몇개인지도 알기 어려움

- 문자열 상수를 사용하는 변형 패턴도 있으나 더 별로임

#
### 2. 열거타입
- 일정 개수의 상수 값을 정의하고 사용하는 타입



