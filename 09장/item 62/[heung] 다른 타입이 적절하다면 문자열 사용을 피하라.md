# 다른 타입이 적절하다면 문자열 사용을 피하라.
> 문자열을 쓰지 않아야 할 사례

<br>

## 1. 문자열은 다른 값 타입을 대신하기에 적합하지 않다.  
기본 타입이든 참조 타입이든 적절한 값 타입이 있다면 그것을 사용하고, 없다면 새로 하나 작성하라. 

<br>

## 2. 문자열은 열거 타입을 대신하기에 적합하지 않다.
상수를 열거할 때는 문자열보다는 열거 타입이 월등히 낫다.

<br>

## 3. 문자열은 혼합 타입을 대신하기에 적합하지 않다.
각 요소를 개별로 접근하려면 문자열을 파싱해야 해서 느리고, 귀찮고, 오류 가능성도 커진다.   
적절한 equals, toString, compareTo 메서드를 제공할 수 없으며, String이 제공하는 기능에만 의존해야 한다.   

=> 전용 클래스를 만드는 편이 낫다. 

<br>

## 4. 문자열은 권한을 표현하기에 적합하지 않다.
권한(capacity)을 문자열로 표현하는 경우가 종종 있다. 예를 들어 스레드 지역변수 기능을 설계한다고 해보자.
```java
public class ThreadLocal {
  private ThreadLocal() { }

  public static void set(String key, Object value);

  public static Object get(String key);
}
```
이 방식의 문제는 스레드 구분용 문자열 키가 전역 이름공간에서 공유된다는 점이다. 보안도 취약하다. 

<br>

문자열 대신 위조할 수 없는 키를 사용하면 해결된다. 이 키를 권한(capacity)이라고 한다. 
```java
public class ThreadLocal {
  private ThreadLocal() { }

  public static class Key {
    Key() { }
  }

  public static Key getKey() {
    return new Key();
  }

  public static void set(String key, Object value);
  public static Object get(String key);
}
```

<br>

개선점1 : set, get은 정적 메서드일 이유가 없으니 Key 클래스의 인스턴스 메서드로 바꾸자. ThreadLocal 자체로 Key가 된다. 
```java
public final class ThreadLocal {
  private ThreadLocal();
  public void set(Object value);
  public Object get();
}
```

<br>

개선점2 : ThreadLocal을 매개변수화 타입으로 선언한다. (타입안전!)
```java
public final class ThreadLocal<T> {
  private ThreadLocal();
  public void set(T value);
  public T get();
}
```

<br>

## 5. 핵심 정리
*  더 적합한 데이터 타입이 있거나 새로 작성할 수 있다면, 문자열을 쓰고 싶은 유혹을 뿌리쳐라.

<br>
