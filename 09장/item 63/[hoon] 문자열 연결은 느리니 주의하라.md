## 1. 들어가기

Java에서 두 문자열을 연결할 때 어떻게 사용하시나요?

보통 편히 사용하는 방법은 문자열 연결 연산자(+)를 사용하는 것입니다.

바로 다음 예시와 같이 말이죠.

## 2. 문자열 연결의 잘못된 예시

```java
   public String statement() {
      String result = "";
      for (String word : words)
         result += word;

      return result;
   }
```

한 줄짜리 작은 문자열은 괜찮지만 여러 문자열을 연결하는 경우, 성능 저하를 일으킬 수 있습니다.

문자열 연결 연산자로 문자열 n개를 연결하는 시간은 n²에 비례합니다.

그 이유는 문자열은 불변이기에 두 문자열을 연결하는 경우 양쪽 내용을 모두 복사해야 하기 때문입니다.

그럼 대안으로는 어떤 방법이 있을까요?

한 번쯤 들어본 StringBuilder를 통해서 해결할 수 있습니다.

## 3. 문자열 연결의 올바른 예시

```java
   public String statement() {
      StringBuilder result = new StringBuilder(wordAllLength);
      for (String word : words)
         result.append(word);

      return result;
   }
```

StringBuilder를 사용하면 문자열 n개를 연결하는 시간은 n에 비례합니다.

그렇기에 단어 개수가 늘어날수록 성능은 훨씬 좋아질 것입니다.

## 4. 정리

이번 포스트는 StringBuilder를 통한 올바른 문자열 연결 예시를 알아보았습니다.

문자열을 연결할 때는 문자열 연결 연산자(+) 대신 StringBuilder를 사용합시다.