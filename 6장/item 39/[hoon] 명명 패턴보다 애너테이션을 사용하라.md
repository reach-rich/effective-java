## 1. 들어가기

전통적으로 도구나 프레임워크가 특별히 다뤄야 할 프로그램 요소에는 명명 패턴을 적용했었습니다.

예를 들면, Junit 3은 테스트 메서드 이름을 `test`로 시작하게 했습니다.

이렇게 명명 패턴을 적용하면 어떤 문제가 발생할까요?

## 2. 명명 패턴의 단점

1. 오타를 허용하지 않는다.

   개발자가 실수로 `testSafetyOverride` 대신 `tsetSafetyOverride` 라 이름을 짓는다면

   Junit 3은 이 메서드를 무시하고 지나치기 때문에 개발자는 테스트가 실패하지 않았기 때문에

   모든 테스트 메서드가 통과했다고 생각할 수 있습니다.

   <br>

2. 올바른 프로그램 요소에서만 사용되리라 보증할 방법이 없다.

   예를 들어 개발자가 메서드가 아닌 클래스에 `test` 키워드를 적용한다면

   테스트 메서드를 수행하길 기대하겠지만, Junit 3는 개발자가 의도한 테스트를 수행하지 않습니다.

   <br>

3. 프로그램 요소를 매개변수로 전달할 방법이 없다.

   예를 들어 특정 예외를 던져야 성공하는 테스트가 있다고 가정해봅시다.

   이럴 경우, 특정 예외 타입을 테스트의 매개변수로 전달해야 하는 상황인데

   명명 패턴으로는 방법이 없습니다.

## 3. 애너테이션의 사용

애너테이션은 명명 패턴의 단점을 모두 해결해주는 개념으로 Junit 4부터 전면 도입되었습니다.

그럼 앞에서 설명한 명명 패턴의 단점을 애너테이션으로 해결해봅시다.

1. 마커(marker) 애너테이션의 사용

   마커 애너테이션은 아무 매개변수 없이 단순히 마킹에 사용하는 애너테이션입니다.

   ```java
    import java.lang.annotation.*;

    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
    public @interface test {

    }
   ```

   애너테이션 내부를 보면 Retention과 Target 애너테이션이 있는데

   이처럼 애너테이션 선언에 다는 애너테이션을 "메타 애너테이션" 이라 합니다.

   > * Retention 애너테이션
   >
   >   🔸 애너테이션의 라이프사이클을 지정
   >
   >   🔹 RetentionPolicy.SOURCE : 소스 코드(.java)까지 남아있는다.<br>
   >   🔹 RetentionPolicy.CLASS : 클래스 파일(.class)까지 남아있는다.(=바이트 코드)<br>
   >   🔹 RetentionPolicy.RUNTIME : 런타임까지 남아있는다.(=사실상 안 사라진다.)

   > * Target 애너테이션
   >
   >   🔸애너테이션의 위치 지정
   >
   >   🔹 ElementType.PACKAGE : 패키지 선언<br>
   >   🔹 ElementType.TYPE : 타입 선언<br>
   >   🔹 ElementType.ANNOTATION_TYPE : 어노테이션 타입 선언<br>
   >   🔹 ElementType.CONSTRUCTOR : 생성자 선언<br>
   >   🔹 ElementType.FIELD : 멤버 변수 선언<br>
   >   🔹 ElementType.LOCAL_VARIABLE : 지역 변수 선언<br>
   >   🔹 ElementType.METHOD : 메서드 선언<br>
   >   🔹 ElementType.PARAMETER : 전달인자 선언<br>
   >   🔹 ElementType.TYPE_PARAMETER : 전달인자 타입 선언<br>
   >   🔹 ElementType.TYPE_USE : 타입 선언
   
   마커 애너테이션은 다음과 같이 사용할 수 있습니다.

   ```java
    public class Sample {
      /* 성공 테스트 */
      @Test
      public static void successTest() {}
      
      /* 실패 테스트 */
      @Test
      public static void failTest() {
        throw new RuntimeException("실패");
      }      


      /* 잘못 사용한 예 */
      @Test
      public wrongTest1() {}                // 정적 메서드 x

      public static void wrongTest2() {}    // Test 애너테이션 x
    }
   ```

2. 하나의 매개변수를 받는 애너테이션의 사용

   ```java
    import java.lang.annotation.*;

    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
    public @interface ExceptionTest {
      Class<? extends Throwable> value();
    }
   ```

   이 애너테이션의 매개변수 타입은 `Class<? extends Throwable>` 으로 명시되어 있는데

   즉, Throwable을 확장한 클래스의 Class 객체라는 뜻으로 모든 예외 타입을 수용한다는 의미입니다.

   하나의 매개변수를 받을 수 있는 애너테이션은 다음과 같이 사용할 수 있습니다.

   ```java
    public class Sample2 {
      /* 성공 테스트 */
      @ExceptionTest(ArithmeticException.class)
      public static void success() {
        int i = 0;
        i = i / i;
      }

      /* 실패 테스트 (다른 예외 발생) */
      @ExceptionTest(ArithmeticException.class)
      public static void fail1() {
        int[] a = new int[0];
        int i = a[1];
      }

      /* 실패 테스트 (예외 발생 x) */
      @ExceptionTest(ArithmeticException.class)
      public static void fail2() {
        
      }
    }
   ```

3. 여러 매개변수를 받는 애너테이션의 사용

   여러 매개변수를 받을 때는 배열을 사용합니다.

   ```java
    import java.lang.annotation.*;

    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
    public @interface ExceptionTest {
      Class<? extends Throwable>[] value();
    }
   ```

   그리고 여러 매개변수를 받을 수 있는 애너테이션은 다음과 같이 사용할 수 있습니다.

   ```java
    public class Sample3 {
      @ExceptionTest({ ArithmeticException.class, NullPointerException.class })
      public static void success() {
        List<String> list = null;
        list.get(0);
      }
    }
   ```

4. Java 8에서의 여러 매개변수를 받는 애너테이션의 사용

   배열 매개변수를 사용하는 대신 애너테이션에 @Repeatable 메타 애너테이션을 달 수 있습니다.

   하지만, 여기서 끝이 아니라 컨테이너 애너테이션을 하나 더 정의하고,

   @Repeatable에 컨테이너 애너테이션의 class 객체를 매개변수로 전달해야 합니다.

   ```java
    import java.lang.annotation.*;

    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
    @Repeatable(ExceptionTestContainer.class)
    public @interface ExceptionTest {
      Class<? extends Throwable> value();
    }

    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
    public @interface ExceptionTestContainer {
      ExceptionTest[] value();
    }
   ```

   ```java
    public class Sample4 {
      @ExceptionTest(ArithmeticException.class)
      @ExceptionTest(NullPointerException.class)
      public static void success() {
        List<String> list = null;
        list.get(0);
      }
    }
   ```

## 4. 정리

이번 포스트는 명명 패턴의 단점을 애너테이션으로 해결하는 방법을 알아보았습니다.

대부분의 경우 애너테이션이 명명 패턴보다 낫기 때문에

애너테이션으로 할 수 있는 일을 굳이 명명 패턴으로 처리할 이유는 없으므로 애너테이션을 사용합시다.