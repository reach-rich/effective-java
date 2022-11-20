# 명명 패턴보다 애너테이션을 사용하라

전통적으로 프레임워크가 특별히 다뤄야하느 요소에는 명명패턴을 적용

JUnit은 버전 3까지 테스트 메서드 이름을 test로 시작

#
### 1. 명명 패턴의 단점

- 오타에 민감 (tesstCode 하면 실행 X)
- 올바르게 사용될거라느 보장이 없음 (Class 이름을 TestClass로 하는 경우)
- 프로스램 요소를 매개변수로 전달할 방법이 없음

#
### 2. 명명 패턴의 대안, 어노테이션
Test 어노테이션 예시 (예외 발생 시 테스트 실패로 처리)

```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {}
```
- 어노테이션 선언에 어노테이션 달려있는 것은 meta-annotation 
- @Retension: 어노테이션 선언된 어노테이션 @Test가 런타임에도 유지되어야 한다는 표시
- @Target (ElementType.METHOD): 어노테이션 선언된 어노테이션 메서드 선언에 사용되어야 함을 표기


#
### 3. 매개변수를 받는 어노테이션 타입

```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable> value();//annotation의 매개변수
}
```

<br>

__배열 매개변수를 받는 어노테이션 타입__

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Exception>[] value();
}
```

```java
@ExceptionTest({ IndexOutOfBoundsException.class, NullPointerException.class }) //배열 매개변수 사용법
public static void test() { ... }
```

<br>

__@Repeatable을 사용한 어노테이션 타입__

컨테이너 어노테이션 타입을 사용해 배열로 선언하는 대신 어노테이션을 여러번 사용 가능

```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
    ExceptionTest[] value();
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}
```

```java
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class) //여러번 사용 가능
public static void test() { ... }
```










