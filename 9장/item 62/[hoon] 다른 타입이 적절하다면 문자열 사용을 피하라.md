## 1. 들어가기

텍스트를 표현하도록 설계된 문자열은 개발에서 흔히 사용됩니다.

하지만 문자열을 본래의 의도가 아닌 잘못 사용하는 예도 많은데요.

이번에는 문자열을 잘못 사용한 예를 알아보겠습니다.

## 2. 문자열을 잘못 사용한 예

1. 다른 값 타입 대신 사용하는 경우

   많은 개발자들은 데이터를 처리할 때 주로 문자열을 사용합니다.

   하지만 데이터가 텍스트일 때만 문자열로 처리하는 것이 좋습니다.

   받은 데이터가 수치형이라면 `int`, `float`, `BigInteger` 등 적당한 타입을 사용해야 하고,

   예/아니오와 같은 질문의 답이라면 `boolean`을 사용해야 합니다.

   <br>

2. 열거 타입을 대신 사용하는 경우

   상수를 문자열로 작성하는 경우 다음과 같이 예상치 못한 결과가 발생할 수 있습니다.

   ```java
      public String getFruit(String fruit) {
         switch (fruit) {
            case "Apple":
               return "사과";
               ...

            default:
               return "???";
         }
      }

      // 예상치 못한 결과 발생
      getFruit("apple"); // ???
      getFruit("Appla"); // ???
   ```

   따라서 문자열 대신 열거 타입을 정의해 사용해야 합니다.

   ```java
      public enum Fruit {
         APPLE, GRAPE
      }

      public String getFruit(Fruit fruit) {
         switch (fruit) {
            case APPLE:
               return "사과";
               ...

            default:
               return "???";
         }
      }

      // 예상치 못한 결과 발생
      getFruit(apple); // 컴파일 오류
      getFruit(APPLA); // 컴파일 오류
   ```

   <br>

3. 혼합 타입 대신 사용하는 경우

   여러 요소가 혼합된 데이터를 문자열로 사용하는 경우가 종종 있습니다.

   ```java
      String compoundKey = className + "#" + i.next();
   ```

   위 예시는 단점이 많습니다.
   
   양쪽 요소 중 문자 #이 쓰였다면 혼란스러운 결과를 초래하기도 하고

   각 요소를 파싱해야해 느리고 귀찮고 오류 가능성이 커집니다.

   또한, String이 제공하는 기능에만 의존해야 합니다.

   따라서 전용 클래스를 별도로 만들어 사용하는 편이 낫습니다.

   <br>

4. 문자열로 권한을 표현하는 경우

   `ThreadLocal` 클래스에서 문자열을 사용했다고 가정해봅시다.

   ```java
      public class ThreadLocal {
         private ThreadLocal() {}

         // 현 스레드의 값을 키로 구분해 저장한다.
         public static void set(String key, Object value);

         // (키가 가리키는) 현 스레드의 값을 반환한다.
         public static Object get(String key);
      }
   ```

   이 방식의 문제는 스레드 구분용 키가 전역 이름공간에서 공유된다는 점입니다.

   만약, 두 클라이언트가 서로 소통하지 못해 같은 문자열을 사용하게 된다면
   
   의도치 않게 같은 변수를 공유하게 됩니다.

   악의적인 클라이언트라면 의도적으로 같은 키를 사용해 다른 클라이언트 값을 가져올 것입니다.

   따라서 문자열 대신 위조할 수 없는 키를 사용해야 합니다.

   ```java
      public class ThreadLocal<T> {
         private ThreadLocal() {}

         public static class Key {
            Key();
         }

         public static void set(Key key, T value);
         public static T get(Key key);
      }
   ```



   set과 get은 더이상 정적 메서드일 이유가 없으므로 Key 클래스의 인스턴스 메서드로 변경합니다.

   그렇게 되면 ThreadLocal은 그 자체가 스레드 지역변수가 되므로 다음과 같이 변경할 수 있습니다.

   ```java
      public final class ThreadLocal<T> {
         public ThreadLocal();
         public void set(T value);
         public T get();
      }
   ```

## 3. 정리

이번 포스트는 문자열 타입을 잘못 사용하는 예를 알아보았습니다.

문자열은 편하지만 잘못 사용하면 번거롭고, 덜 유연하고, 느리며 오류가능성이 큽니다.

그러므로 적절한 다른 타입이 있다면 문자열 타입 사용을 지양합시다.