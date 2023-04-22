## 1. 들어가기

Java는 다중정의(Overloading, 오버로딩)를 제공합니다.

다중정의란, 서로 다른 시그니처를 갖는 여러 메소드를 같은 이름으로 정의하는 것인데

다중정의를 사용하면 하나의 이름만 사용하기 때문에 기억하기도 쉽고 오류 가능성을 줄일 수 있습니다.

하지만 다중정의를 잘못 사용하면 예기치 못한 오류 발생 가능성이 있습니다.

어떻게 사용하면 문제가 되는지 한번 살펴봅시다!

## 2. 잘못 사용한 다중정의

다음은 컬렉션을 집합, 리스트, 그 외로 구분하기 위한 프로그램입니다.

```java
   public class CollectionClassifier {
      public static String classify(Set<?> s) {
         return "집합";
      }

      public static String classify(List<?> list) {
         return "리스트";
      }

      public static String classify(Collection<?> c) {
         return "그 외";
      }
   }

   public static void main(String[] args) {
        Collection<?>[] collections = {
                new HashSet<String>(),
                new ArrayList<BigInteger>(),
                new HashMap<String, String>().values()
        };

        for (Collection<?> c : collections)
            System.out.println(classify(c));
    }
```

```
=== 기대값 ===
   집합
   리스트
   그 외

=== 결과값 ===
   그 외
   그 외
   그 외
```

`collections` 배열에 집합, 리스트, Map이 들어있기 때문에 `집합`, `리스트`, `그 외`로 출력하리라 생각했지만,

실제로는 `그 외`만 세 번 연달아 출력합니다. 

왜 그런걸까요?? 🤔

바로, 다중정의의 메서드 호출은 런타임이 아닌 컴파일타임에 정해지기 때문입니다.

그래서 컴파일타임에는 반복문 내부의 c는 항상 `Collection<?>` 타입이므로

항상 세 번째 메서드인 `classify(Collection<?>)`만 호출하는 것입니다.

그럼 이런 문제는 어떻게 해결할 수 있을까요?

## 3. 잘못 사용한 다중정의 해결 방법

다중정의는 프로그래머가 기대한 대로 동작하지 않을 수 있습니다.

그러므로 헷갈릴 수 있는 코드는 작성하지 않는 것이 좋습니다.

그래서 다중정의 코드를 다음과 같이 해결할 수 있습니다.

* 모든 classify 메서드를 하나로 합친 후 instanceof로 명시적으로 검사한다.

```java
   public static String classify(Collection<?> c) {
      return c instanceof Set  ? "집합" :
            c instanceof List ? "리스트" : "그 외";
   }
```

<br>

* 메서드 이름을 다르게 지어준다.

```java
   public class CollectionClassifier {
      public static String classifySet(Set<?> s) {
         return "집합";
      }

      public static String classifyList(List<?> list) {
         return "리스트";
      }

      public static String classify(Collection<?> c) {
         return "그 외";
      }
   }
```

<br>

* 인수 포워드 방식을 사용한다.

   어쩔 수 없이 다중정의를 사용해야 한다면 인수 포워드 방식을 고려해봅시다.

   인수 포워드 방식은 두 메서드가 동일한 일을 하도록 보장할 수 있습니다.

   ```java
      public boolean contentEquals(StringBuffer sb) {
         return contentEquals((CharSequence) sb);
      }
   ```

## 4. 다중정의를 사용할 때 주의해야할 점

이번에는 다중정의를 사용한다면 어떤 부분을 주의해야할 지 알아보겠습니다.

* 매개변수를 근본적으로 다르게 한다.

   근본적으로 다르다는 것은 두 타입을 서로 형변환할 수 없는 것을 말합니다.

   ```java
      public class SetList {
         public static void main(String[] args) {
            Set<Integer> set = new TreeSet<>();
            List<Integer> list = new ArrayList<>();

            for (int i=-3; i<3; i++) {
               set.add(i);
               list.add(i);
            }

            for (int i=0; i<3; i++) {
               set.remove(i);
               list.remove(i);
            }

            System.out.println(set);
            System.out.println(list);
         }
      }
   ```

   ```
      [-3, -2, -1]
      [-2, 0, 2]
   ```

   둘다 동일하게 [-3, -2, -1]을 출력하리라 예상했지만 실제로 리스트에서는 [-2, 0, 2]를 출력합니다.

   그 이유는 리스트의 remove 메서드가 int 매개변수와 Object 매개변수로 다중정의되었기 때문입니다.

   그래서 이 문제는 인수를 Integer로 형변환하면 해결됩니다.

   ```java
      for (int i=0; i<3; i++) {
         set.remove(i);
         list.remove((Integer) i);
      }
   ```

* 서로 다른 함수형 인터페이스여도 위치를 다르게 한다.

   ```java
   // 1. Thread의 생성자 호출
      new Thread(System.out::println).start();

   // 2. ExecutorService의 submit 메서드 호출
      ExecutorService exec = Executors.newCachedThreadPool();
      exec.submit(System.out::println);
   ```

   위의 예시는 넘겨진 인수가 `System.out::println`으로 동일하고,

   양쪽 모두 `Runnable`을 받는 형제 메서드를 다중정의하고 있지만 2번 코드만 컴파일 오류가 발생합니다.

   원인은 바로 `submit`과 `println` 모두 다중정의되었기 때문입니다.

   따라서, 비록 서로 다른 함수형 인터페이스라도 인수 위치가 같으면 혼란이 생기므로

   메서드를 다중정의할 때는 서로 다른 함수형 인터페이스라도 같은 위치의 인수로 받아서는 안됩니다.

## 5. 정리

이번 포스트는 다중정의에 대해 알아보았습니다.

프로그래밍 언어가 다중정의를 허용한다고 해서 다중정의를 꼭 활용하라는 의미는 아닙니다.

일반적으로 매개변수 수가 같을 때는 다중정의를 피하는 게 좋습니다.

하지만, 피할 수 없을 경우에는 정확한 다중정의 메서드가 선택되도록 하거나

인수를 포워드해 모든 다중정의 메서드들이 동일하게 동작하도록 구현해야 합니다.