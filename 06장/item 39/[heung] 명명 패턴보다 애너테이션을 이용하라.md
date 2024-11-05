Effective Java의 서른아홉 번째 아이템 "명명 패턴보다 애너테이션을 이용하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 1. 명명 패턴의 단점

전통적으로 도구나 프레임워크가 특별히 다뤄야 할 프로그램 요소에는 딱 구분되는 명명 패턴을 적용해왔다. 예를 들어 테스트 프레임워크인 JUnit은 버전 3 까지 테스트 메서드 이름을 test로 시작하게끔 했다. 효과적인 방법이지만 단점도 크다.

* 오타나가 나면 안 된다.
* 올바른 프로그램 요소에서만 사용되리라 보증할 방법이 없다.
* 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다.



## 2. 애너테이션

애너테이션은 이 모든 문제를 해결해주는 멋진 개념으로, JUnit도 버전 4부터 전면 도입하였다. 

### 예제 1)

Test라는 이름의 애너테이션을 정의한다고 해보자. 자동으로 수행되는 간단한 테스트용 애너테이션으로, 예외가 발생하면 해당 테스트를 실패로 처리한다.

```java
import java.lang.annotation.*;

/**
 * 테스트 메서드임을 선언하는 애너테이션이다.
 * 매개변수 없는 정적 메서드 전용이다.
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
}
```

@Test 애너테이션 타입 선언 자체에도 두 가지의 다른 애너테이션이 달려 있다. 이처럼 애너테이션 선언에 다는 애너테이션을 메타애너테이션이라 한다.

* @Retention : @Test가 런타임에도 유지되어야 한다. 이 애너테이션을 생략하면 테스트 도구는 @Test를 인식할 수 없다.
* @Target : @Test가 반드시 메서드 선언에서만 사용돼야 한다.

> 앞 코드의 메서드 주석에는 "매개변수 없는 정적 메서드 전용이다"라고 쓰여있다. 이 제약을 컴파일러가 강제할 수 있으면 좋겠지만, 그렇게 하려면 적절한 애너테이션 처리기를 직접 구현해야 한다.
>
> 적절한 애너테이션 처리기 없이 인스턴스 메서드나 매개변수가 있는 메서드에 달면 어떻게 될까? 컴파일은 잘 되겠지만, 테스트 토구를 실행할 때 문제가 된다.

<br>

다음은 @Test 애너테이션을 실제 적용한 모습이다. 이와 같은 애너테이션을 "아무 매개변수 없이 단순히 대상에 마킹한다"는 뜻에서 마커 애너테이션이라 한다. 이 애너테이션을 사용하면 프로그래머가 Test 이름에 오타를 내거나 메서드 선언 외의 프로그램 요소에 달면 컴파일 오류를 내준다.

```java
public class Sample {
  @Test public static void m1() {}
  public static void m2() {}
  @Test public static void m3() {
    throw new RuntimeException("실패");
  }
  public static void m4() {}
  @Test public void m5() {}
  public static void m6() {}
  @Test public static void m7() {
    throw new RuntimeException("싪패");
  }
  public static void m8() {}
}
```

@Test 애너테이션을 단 4개의 테스트 메서드 중 1개는 성공, 2개는 실패, 1개는 잘못 사용했다. 그리고 @Test를 붙이지 않은 나머지 4개의 메서드는 테스트 도구가 무시할 것이다.

@Test 애너테이션이 Sample 클래스의 의미에 직접적인 영향을 주지는 않는다. 그저 이 애너테이션에 관심 있는 프로그램에게 추가 정보를 제공할 뿐이다. 더 넒게 이야기하면, 대상 코드의 의미는 그대로 둔 채 그 애너테이션에 관심 있는 도구에서 특별한 처리를 할 기회를 준다. 다음의 RunTests가 바로 그런 도구의 예다.

```java
import java.lang.reflect.*;

public class RunTests {
  public static void main(String[] args) throws Exception {
    int tests = 0;
    int passed = 0;
    Class<?> testClass = Class.forName(args[0]);
    for (Method m : testClass.getDeclaredMethod()) {
      if (m.isAnnotationPresent(Test.class)) {
        tests++;
        try {
          m.invoke(null);
          passed++;
        } catch (InvocationTargetException warppedExc) {
          Throwable exc = wrappedExc.getCause();
          System.out.println(m + " 실패: " + exc);
        } catch (Exception exc) {
          System.out.println("잘못 사용한 @Test: " + m);
        }
      }
    }
    System.out.printf("성공: %d, 실패: %d%n", passed, test - passed);
  }
}
```

테스트 메서드가 예외를 던지면 리플렉션 메커니즘이 InvocationTargetException으로 감싸서 다시 던진다. 그래서 이 프로그램은 InvocationTargetException을 잡아 원래 예외에 담긴 실패 정보를 추출해 출력한다. InvocationTargetException외의 예외가 발생한다면 @Test 애너테이션을 잘못 사용했다는 뜻이다.

다음은 이 RunTests로 Sample을 실행했을 때의 출력 메시지다.

```tex
public static void Sample.m3() faild: RuntimeException: Boom
Invalid @Test: public void Sample.m5()
public static void Sample.m7() faild: RuntimeException: Crash
성공: 1, 실패: 3
```

<br>

### 예제 2)

이제 특정 예외를 던져야만 성공하는 테스트를 지원하도록 해보자.

```java
import java.lang.annotation.*;

/**
 * 명시한 예외를 던져야한 성공하는 테스트 메서드용 애너테이션
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
  Class<? extends Throwable> value();
}
```

이 애너테이션의 매개변수 타입은 Class<? extends Throwable>이다. 여기서의 와일드카드 타입은 많은 의미를 담고 있다. "Throwable을 확장한 클래스의 Class 객체"라는 뜻이며, 따라서 모든 예외(와 오류) 타입을 다 수용한다.

<br>

이 애너테이션을 실제로 활용한 모습이다. class 리터럴은 애너테이션 매개변수의 값으로 사용됐다. 

```java
public class Sample2 {
  @ExceptionTest(ArithmeticException.class)
  public static void m1() { // 성공
    int i = 0;
    i = i / i;
  }
  @ExceptionTest(ArithmeticException.class)
  public static void m2() { // 실패 (다른 예외 발생)
    int[] a = new int[0];
    int i = a[1];
  }
  @ExceptionTest(ArithmeticException.class)
  public static void m3() {} // 실패 (예외가 발생하지 않음)
}
```

<br>

이제 이 애너테이션을 다룰 수 있도록 테스트 도구를 수정해보자.

```java
...
} catch (InvocationTargetException warppedExc) {
  Throwable exc = wrappedExc.getCause();
  Class<? extends Throwable> excType = m.getAnnotation(ExceptionTest.clas).value();
  if (excType.isInstance(exc)) {
    passed++;
  } else {
    System.out.printf("테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s%n", m, excType.getName(), exc);
  }
} catch (Exception exc) {
  System.out.println("잘못 사용한 @ExceptionTest: " + m);
}
```

이 코드는 애너테이션 매개변수의 값을 추출하여 테스트 메서드가 올바른 예외를 던지는지 확인하는 데 사용한다. 단, 해당 예외의 클래스 파일이 컴파일타임에는 존재했으나 런타임에는 존재하지 않을 수는 있다. 이런 경우라면 테스트 러너가 TypeNotPresentException을 던질 것이다.

<br>

### 예제 3)

예외를 여러 개 명시하고 그중 하나가 발생하면 성공하게 만들 수도 있다. 

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
  Class<? extends Throwable>[] value();
}
```

원소가 여럿인 배열을 지정할 때는 다음과 같이 원소들을 중괄호로 감싸고 쉼표로 구분해주기만 하면 된다.

```java
@ExceptionTest({ IndexOutOfBoundsException.class,
               NullPointerException.class })
public static void doublyBad() {
  List<String> list = new ArrayList<>();
  
  list.addAll(5, null);
}
```

<br>

자바 8에서는 여러 개의 값을 받는 애너테이션을 다른 방식으로도 만들 수 있다. 배열 매개변수를 사용하는 대신 애너테이션 @Repeatable 메타애너테이션을 다는 방식이다. @Repeatable을 단 애너테이션은 하나의 프로그램 요소에 여러 번 달 수 있다. 단, 주의할 점이 있다.

* @Repeatable을 단 애너테이션을 반환하는 '컨테이너 애너테이션'을 하나 더 정의하고, @Repeatable에 이 컨테이너 애너테이션의 class 객체를 매개변수로 전달해야 한다.
* 컨테이너 애너테이션은 내부 애너테이션 타입의 배열을 반환하는 value 메서드를 정의해야 한다.
* 컨테이너 애너테이션 타입에는 적절한 보존 정책(@Retention)과 적용 대상(@Target)을 명시해야 한다.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
  Class<? extends Throwable> value();
}
```

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
  ExceptionTest[] value();
}
```

이제 앞서의 배열 방식 대신 반복 가능 애너테이션을 적용해보자.

```java
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class)
public static void doublyBad() { ... }
```

<br>

## 3. 핵심 정리

* 애너테이션으로 할 수 있는 일을 명명 패턴으로 처리할 이유는 없다.
* 다른 프로그래머가 소스코드에 추가 정보를 제공할 수 있는 도구를 만드는 일을 한다면 적당한 애너테이션 타입도 함께 정의해 제공하자.

<br>

## 4. Related Posts

* [한정적 타입 토큰 (Item 33)](https://heung27.github.io/posts/item-33-%ED%83%80%EC%9E%85-%EC%95%88%EC%A0%84-%EC%9D%B4%EC%A2%85-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88%EB%A5%BC-%EA%B3%A0%EB%A0%A4%ED%95%98%EB%9D%BC/)

