## item 62 다른 타입이 적절하다면 문자열 사용을 피하라

문자열은 텍스트를 표현하도록 설계되었고 자바가 잘 지원해주어 원래 의도하지 않는 용돋로도 쓰이는 경향이 있다. <br>
문자열을 쓰지 말아야할 상황에 대해 알아보자


#
### 1. 리터럴리 문자열이 아니라면 사용하지 말자
- 파일, 네트워크, 키보드 입력을 받을 때 무조건 문자열을 사용하는 것을 지양하는 것이 좋음
- 기본타입, 참조타입을 사용하고 적절한 값 타입이 없다면 하나 새로 작성하기

#
### 2. 열거타입 대신 문자열을 쓰지 말자
- 문자열은 열거 타입을 대신하기에 적합하지 않음

```java
String compoundKey = className + "#" + i.next();
```

- 위와 같은 경우 파싱이 필요해 느려지고 귀찮고 오류 가능성도 커짐
- 전용 클래스를 새로 만드는게 나음 (private 정적 멤버 클래스 item 24)

#
### 3. 문자열로 권한을 표현하지 말자

```java
public class ThreadLocal {
  private ThreadLocal() {} // 객체 생성 불가

  // 현 스레드의 값을 키로 구분해 저장
  public static void set(String key, Object value);

  // (키가 가리키는) 현 스레드 값을 반환
  public static Object get(String key);
}
```

- 구분용 문자열 키가 전역 이름 공간에서 공유됨
- 두 클라이언트가 같은 키를 사용하면 변수가 공유됨
- 악의적인 사용자라면 의도적으로 키 값을 악용해 보안에도 취약

> 문자열 대신 위조할 수 없는 키를 사용하자 -> 권한(capacity)

```java
public class ThreadLocal<T> {
  public ThreadLocal(); 
  public void set(T value);
  public T get();
}
```
- 문자열 기반의 API 문제를 해결하고 키 기반 API보다 빠르고 우아함 (?)

> ThreadLocal : https://javacan.tistory.com/entry/ThreadLocalUsage
