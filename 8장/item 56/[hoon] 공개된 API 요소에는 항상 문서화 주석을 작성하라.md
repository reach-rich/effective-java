## 1. 들어가기

공개 API를 작성할 때는 반드시 문서화 주석을 작성해야 합니다.

Java에서는 JavaDoc이라는 유틸리티가 그 역할을 해주는데

JavaDoc은 문서화 주석이라는 특수한 형태로 기술된 설명을 추려 API 문서로 변환해줍니다.

그럼, JavaDoc에 대해 자세히 알아보겠습니다.

## 2. JavaDoc 사용법

1. `@throws`

   메서드 내에서 발생할 가능성이 있는 모든 예외에 대해 명시하는 태그입니다.

   ```java
    /**
     * @throws IndexOutOfBoundsException if the index is out of range
     */
    E get(int index);
   ```

   <br>

2. `@param`

   메서드의 파라미터에 대한 정보로 매개변수가 뜻하는 값이나 반환값을 명사구로 작성합니다.

   ```java
   /**
    * @param index index of element to return; must be non-negative and less than the size of this size
    */
    E get(int index);
   ```

   <br>

3. `@return`

   메서드의 반환값에 대한 정보로 반환 타입이 void가 아니라면 반환값을 명사구로 작성합니다.

   ```java
   /**
    * @return the element at the specified position in this list
    */
    E get(int index);
   ```

   <br>

4. `@code`

   코드를 명시하는 태그로 코드용 폰트로 렌더링해주고, HTML 요소나 다른 javaDoc 태그를 무시합니다.

   ```java
    /**
     * @throws IndexOutOfBoundsException if the index is out of range
     *         ({@code index < 0 || index >= this.size()})
     */
     E get(int index);
   ```

   만약, 주석에 여러 줄로 된 코드 예시를 넣으려면 `{@code}` 태그를 다시 \<pre> 태그로 감싸면 됩니다.

   ```java
    /**
     * @throws IndexOutOfBoundsException if the index is out of range
     *         (<pre>{@code index < 0 || index >= this.size()}</pre>)
     */
     E get(int index);
   ```

   <br>

5. `@implSec`

   상속용 클래스를 위한 자기사용 패턴용 코드로 해당 메서드와 하위 클래스 사이의 계약을 설명합니다.

   ```java
    /**
     * @implSepc
     * This implementation returns {@code this.size() == 0}.
     * 
     * @return true if this collection is empty.
     */
     public boolean isEmpty();
   ```

   위 태그 사용 시 주의해야할 점은 아래 명령어를 JavaDoc 명령줄에 추가하지 않으면 태그가 무시됩니다.
   
   `-tag "implSpec:a:Implementation Requirements:"`

   <br>

6. `@literal`

   HTML 마크업이나 JavaDoc 태그를 무시하는 태그입니다.

   앞서 `@{code}` 태그와 유사하지만 코드 폰트로 렌더링하지 않는다는 차이가 있습니다.

   ```java
   /**
    * A geometric series converges if {@literal |r| < 1}.
    */
   ```

   <br>

7. 요약 설명

   각 문서화 주석의 첫 번째 문장은 해당 요소의 요약 설명으로 간주됩니다.

   ```java
    /**
     * Returns the element at the specified position in this list.  (요약 설명 부분)
     *
     * @param index index of element to return; must be non-negative and less than the size of this size
     * @return the element at the specified position in this list
     * @throws IndexOutOfBoundsException if the index is out of range
     */
     E get(int index);
   ```

   요약 설명이 끝나는 판단 기준은 `<마침표> <공백> <다음 문장 시작>` 패턴의 `<마침표>`까지 입니다.

   여기서 `<공백>`은 스페이스, 탭, 줄바꿈이며, `<다음 문장 시작>`은 소문자가 아닌 문자입니다.

   주의해야할 점은 다음과 같은 경우입니다.

   `Mrs. Peacock`

   `Mrs` 다음에 마침표와 공백이 나오고 다음 단어의 시작이 `P`이므로 `Mrs`까지만 요약 설명이 됩니다.

   이런 경우엔 `Mrs. Peacock`를 `@literal`로 감싸줌으로써 해결됩니다.

   ```java
    /**
     * A suspect, such as Colonel Mustard or {@literal Mrs. Peacock}.
     */
     public class Suspect{ ... }
   ```

   Java 10부터는 `{@summary}`라는 요약 설명 전용 태그가 추가되어 다음과 같이 사용할 수 있습니다.

   ```java
    /**
     * {@summary A suspect, such as Colonel Mustard or Mrs. Peacock.}
     */
     public class Suspect{ ... }
   ```

   <br>

   한편, 요약 설명은 요소에 따라 문장으로 작성하거나 문장이 아닌 구로 작성해야 하는 경우가 있는데

   메서드와 생성자의 요약 설명은 동작을 설명하는 주어가 없는 동사구로 작성해야 합니다.

   ```java
    /**
     * Constructs an empty list with the specified initial capacity.
     */
     ArrayList(int initialCapacity)
   ```

   반면 클래스, 인터페이스, 필드의 경우는 대상을 설명하는 명사절로 작성해야 합니다.

   ```java
    /**
     * An Instantaneous point on the time-line. 
     */
     public class Instant
   ```

   <br>

8. `@index`

   Java 9부터는 JavaDoc이 생성한 HTML 문서에 검색(색인) 기능이 추가되었습니다.

   `{@index}`태그를 사용해 원하는 용어를 색인화할 수 있습니다.

   ```java
    /**
     * This method complies with the {@index IEEE 754} standard.
     */
   ```

   <br>

9. 제네릭 타입 또는 제네릭 메서드 문서화

   제네릭 타입이나 제네릭 메서드를 문서화할 때는 모든 타입 매개변수에 주석을 작성해야 합니다.

   ```java
    /**
     * @param <K> the type of keys maintained by this map
     * @param <V> the type of mapped values
     */
     public interface Map<K, V> { ... }
   ```

   <br>

10. 열거 타입 문서화

    열거타입을 문서화할 때는 모든 상수들에 주석을 작성해야 합니다.

    이는 열거 타입 자체와 그 열거 타입의 public 메서드도 마찬가지입니다.

    ```java
      /**
       * An instrument section of a symphony orchestra
       */
       public enum OrchestraSection {
          /** WoodWinds, such as flute, clarinet and oboe */
          WOODWIND,

          /** Brass instruments, such as french horn and trumper */
          BRASS,

          /** Percussion instruments, such as timpani, cymbals */
          PERCUSSION,

          /** Stringed instruments, such as violin and cello */
          STRING
       }
    ```

    <br>

11. 애너테이션 타입 문서화

    애너테이션 타입은 모든 멤버들에 주석을 작성해야 합니다.

    이때, 필드 설명은 명사구, 요약 설명은 동사구로 작성합니다.

    ```java
      /**
       * Indicates that the annotated method is a test method that 
       * must throw the designated exception to pass
       */
       @Retention(RetentionPolicy.RUNTIME)
       @Target(ElementType.METHOD)
       public @interface ExceptionTest {

          /**
           *  The exception that the annotated test method must throw
           *  in order to pass. (The test is permitted to throw any subtype
           *  of the type described by this class object.)
           */
           Class<? extends Throwable> value();
       }
    ```

    <br>

12. 패키지 문서화

    패키지를 설명하는 문서화 주석은 package-info.java 파일에 작성합니다.

    이 파일은 반드시 패키지 선언을 반드시 포함해야 하며
    
    패키지 선언 관련 애너테이션을 추가로 포함할 수 있습니다.

    <br>

13. 모듈 시스템 문서화

    Java 9부터 모듈 시스템이 추가되었습니다.

    모듈 시스템의 문서화는 module-info.java 파일에 작성합니다.

14. `@inheritDoc`

    상위 타입의 문서화 주석 일부를 상속할 수 있는 태그로

    비슷한 주석을 여러 개 유지보수하는 부담을 줄일 수 있는 장점이 있지만

    사용하기 까다롭고 제약도 있다는 단점도 있습니다.

## 3. 주의사항

1. 클래스 혹은 정적 메서드의 스레드 안전 수준을 반드시 API 설명에 포함해야 한다.

2. 직렬화할 수 있는 클래스라면 직렬화 형태도 API 설명에 기술해야 한다.

3. 문서화 주석 외 별도의 설명이 필요한 경우, 설명 문서가 존재한다면 해당 링크를 제공한다.

## 4. 정리

이번 포스트는 문서화 주석 사용법에 대해 알아보았습니다.

공개 API를 만들 때는 반드시 문서화 주석을 작성해야 합니다.

작성 시, 표준 규약을 일관되게 지키고 문서화 주석에 임의의 HTML 태그도 사용할 수 있음을 기억합시다.