## 1. 들어가기

개발을 하면서 똑같은 기능의 객체를 여러 번 생성해본 경험이 있나요?

여러 개 생성하는 것이 효율적인 경우도 있을 수 있으나

대부분 같은 객체를 여러 개 생성하는 경우, 성능에 문제가 있을 수 있습니다.

그럼 불필요한 객체를 생성하는 예시들을 알아봅시다.

## 2. 불필요한 객체 생성 예시

먼저, 첫 번째로 볼 예시는 제가 이번 프로젝트에서 작성했던 코드입니다.

보안 상 속성의 이름은 수정했습니다.

```java
  // 필수 속성
  List<String> requiredProperty = Arrays.asList("property1", "property2", "property3", "property4", "property5");

  // Convert DTO to HashMap
  Map<String, Object> registForm = new ObjectMapper().convertValue(bdPartnerDto, HashMap.class);

  // 해당 속성의 값이 null이거나 빈 값이면 false 반환
  if(requiredProperty.stream()
      .anyMatch(property -> !StringUtils.hasText((String)registForm.get(property)))
    )
      return false;
```

해당 코드는 폼 데이터 등록 전 필수 속성 체크 로직입니다.

이 코드 자체에는 문제가 없지만, 문제는 해당 코드를 여러 군데에서 사용했다는 점입니다.

매번 필수 속성을 체크할 때마다 ObjectMapper 객체를 생성해서 사용했는데

ObjectMapper는 생성 비용이 아주 비싸 성능이 중요한 상황에서 반복해 사용하기엔 적합하지 않습니다.

그렇다면 해당 코드를 어떻게 변경할 수 있을까요?

Item 3에서 알아봤던 Singleton으로 ObjectMapper를 만드는 방법이 있습니다.

```java
  public static final ObjectMapper OBJECTMAPPER = new ObjectMapper();
```

objectMapper를 singleton으로 선언하고

첫 번째 예시의 두 번째 줄 코드를 다음과 같이 변경하면 됩니다.

```java
  Map<String, Object> registForm = OBJECTMAPPER.convertValue(bdPartnerDto, HashMap.class);
```

이렇게 개선하면 objectMapper가 빈번히 호출되는 상황에서 성능을 상당히 끌어올릴 수 있게 됩니다.

<hr>
두 번째로 볼 예시는 오토박싱(Auto Boxing) 입니다.

Auto Boxing에 대해서 잠깐 설명하면,

Java에는 int와 같은 기본 타입이 있는데 이 기본 타입들은 Integer와 같은 Wrapper 클래스를 가집니다.

기본 타입에서 Wrapper 클래스로 자동 변환이 이루어지는 것을 Auto Boxing 이라고 하는데

자동 변환이 이루어지지 않는다면 다음과 같이 프로그래머가 타입을 명시해줘야 합니다.

```java
  int from = 12;
  // 기본 타입 -> Wrapper 클래스 명시적 변환
  Integer to = (Integer)from;
```

하지만, Java는 자동변환이 가능하므로 다음과 같이 작성을 해도 무방합니다.

```java
  int from = 12;
  // 기본 타입 -> Wrapper 클래스 묵시적 변환
  Integer to = from;
```

그럼 다시 돌아가서, 불필요한 객체 생성이 되는 Auto Boxing 예시를 보겠습니다.

```java
  private static long sum() {
    Long sum = 0L;
    for(long i=0; i<=Integer.MAX_VALUE; i++)
      sum += i;
    
    return sum;
  }
```

해당 예시는 0부터 Integer의 Max 범위까지 모든 수를 더하는 로직입니다.

프로그램이 정확한 답을 내기는 하지만,

특정 한 줄 때문에 정상 구현했을 때보다 성능이 떨어지게 됩니다.

혹시 찾으셨나요? 🤔🤔

못 찾으신 분은 상심하지 마세요...

저도 한 번에 찾지 못했습니다.

성능에 문제가 되는 부분은 바로, 첫 번째 줄입니다.

Wrapper 클래스인 Long으로 선언했기 때문에 불필요한 Long 인스턴스가 약 21억 개가 생성된 것입니다.

그럼, 이런 의심이 들 수 있습니다.

> 에이, 그거 하나에 성능이 차이가 난다고?

성능을 직접 비교해볼까요?

```java
  // Wrapper 클래스 변수 사용
  private static long wrapperSum() {
    Long sum = 0L;
    for(long i=0; i<=Integer.MAX_VALUE; i++)
        sum += i;

    return sum;
  }

  // Primitive 변수 사용
  private static long primitiveSum() {
    long sum = 0L;
    for(long i=0; i<=Integer.MAX_VALUE; i++)
        sum += i;

    return sum;
  }

  public static void main(String[] args) {
    // Wrapper 클래스 측정
    long startMillisWrapper = System.currentTimeMillis();
    wrapperSum();
    long endMillisWrapper = System.currentTimeMillis();

    // Primitive 타입 측정
    long startMillisPrimitive = System.currentTimeMillis();
    primitiveSum();
    long endMillisPrimitive = System.currentTimeMillis();

    // 측정 결과
    System.out.println("<측정 결과>");
    System.out.println("wrapperSum: " + (endMillisWrapper - startMillisWrapper) + " milliSecond");
    System.out.println("primitiveSum: " + (endMillisPrimitive - startMillisPrimitive) + " milliSecond");
  }
```

```
  <측정 결과>
  wrapperSum: 4890 milliSecond
  primitiveSum: 678 milliSecond
```

보시다시피 Wrapper 클래스 변수를 사용했을 때는 약 4.8초였는데

기본 타입 변수를 사용했을 때는 약 0.6초로

Wrapper 클래스를 기본 타입으로만 변경해도 약 4.2초 정도 절약할 수 있음을 확인했습니다. 

그럼 교훈은 명확하겠네요!

> 의도치 않은 Auto Boxing이 숨어들지 않도록 주의하자.

## 3. 정리

이번 포스트에서는 불필요한 객체 생성을 한 예시를 알아보았습니다.

오해할 수 있는 부분이라면, 이번 Item을 객체 생성을 피해야 한다라고 생각할 수 있는데

프로그램의 명확성, 간결성, 기능을 위해서 추가로 객체를 생성하는 것은

일반적으로 좋은 일입니다.

그러나 생성 비용이 비싼 객체를 여러 번 사용한다든지

혹은, 의도치 않은 Auto Boxing은 지양해야 할 부분입니다.

이번 포스트를 참조해 객체 생성과 캐싱을 적절히 사용한 효율적인 코드를 생산하면 좋겠습니다.