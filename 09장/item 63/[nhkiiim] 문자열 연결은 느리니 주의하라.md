## 🧤 문자열 연결은 느리니 주의하라

- 문자열 연산자(+) 는 여러 문자열을 하나로 합쳐주는 편리한 수단!
  
- 한줄짜리 출력 값 or 작고 크기가 고정된 객체의 문자열에는 괜찮아
- 킹치만.. 본격적으로 사용하기 시작한다면.. 성능 저하가 올 거야..

  <br>

> 🚨  문자열 연결 연산자로 문자열 n개를 잇는 시간은 n2에 비례한다는 사아실...

    왜냐면 문자열은 불변 아이템이라서 두 문자열을 이을 때 양쪽의 내용을 모두 복사해야하거든..

#
### 🏓 문자열 연결 예시 - 느리다!
```java
public String statement() {
  String result = "";

  for (int i = 0; i < numItems(); i++) {
    result += lineForItem(i); // 문자열 연결
  }
}
```

- 대박 짱 느려.. 성능을 포기하고 싶지 않다면..!! StringBuilder를 사용하도록 해 !!

#
### 🏓 StringBuilder 사용하기
```java
public String statement2() {
  StringBuilder b = new StringBuilder(numItems() * LINE_WIDTH);

  for (int i = 0; i < numItems(); i++) {
    b.append(lineForItem(i));
  }

  return b.toString();
}
```
- 자바 6 이후 문자열 연결 성능을 다방면으로 개선했지만, 여전히 StringBuilder가 낫다
