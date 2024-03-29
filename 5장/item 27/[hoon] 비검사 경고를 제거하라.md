## 1. 들어가기

제네릭 사용에 의해 발생하는 컴파일러 경고는 다양합니다.

다양한 경고 중 비검사(unchecked) 경고는 쉽게 제거할 수 있습니다.

여기서 비검사 경고란 무엇일까요?

## 2. 비검사 경고

비검사 경고란, ClassCastException이 발생할 수 있다는 것을 의미합니다.

예시를 통해 알아보겠습니다.

```java
  Set<String> exaltation = new HashSet();
```

위의 코드를 `-Xlint:unchecked` 옵션을 줘서 컴파일하게 되면 컴파일러는 다음의 경고를 보여줍니다.

```
  Main.java:10: warning: [unchecked] unchecked conversion
        Set<String> exaltation = new HashSet();
                                 ^
  required: Set<String>
  found:    HashSet
```

해당 경고를 보고 개발자는 컴파일러가 알려준대로 수정하면 경고가 사라집니다.

```java
  Set<String> exaltation = new HashSet<String>();
```

💡 **결론**

> 비검사 경고를 모두 제거한다면 그 코드는 타입 안전성이 보장된다.

## 3. SuppressWarnings 어노테이션

**🤔 SuppressWarnings 어노테이션이 뭐지?**

Java의 SuppressWarnings 어노테이션은 경고를 제외할 때 사용합니다.

그리고 다음의 옵션을 가집니다.

1. all : 모든 경고를 억제
2. cast : 캐스트 연산자 관련 경고 억제
3. dep-ann : 사용하지 말아야 할 주석 관련 경고 억제
4. deprecation : 사용하지 말아야 할 메소드 관련 경고 억제
5. fallthrough : switch문에서의 break 누락 관련 경고 억제
6. finally : 반환하지 않는 finally 블럭 관련 경고 억제
7. null : null 분석 관련 경고 억제
8. rawtypes : 제네릭을 사용하는 클래스 매개 변수가 불특정일 때의 경고 억제
9. unchecked : 검증되지 않은 연산자 관련 경고 억제
10. unused : 사용하지 않는 코드 관련 경고 억제

**🤔 언제 사용해야할까?**

만약, 경고를 제거할 수는 없지만 타입이 안전하다고 확신할 수 있다면

SuppressWarnings 어노테이션의 unchecked 옵션을 사용할 수 있습니다.

하지만, 이는 경고를 숨겨주는 것이지, 런타임 시에 여전히 ClassCastException 발생 가능성이 존재합니다.

그렇기 때문에 타입 안전성이 보장되는 곳에서만 사용합시다.

**🤔 어디에 선언해야할까?**

SuppressWarnings 어노테이션은 개별 지역변수 선언부터 클래스 전체까지 어떤 선언에 달 수 있습니다.

자칫 심각한 경고를 놓칠 수 있으니 가능한 좁은 범위에 사용해야 합니다.

## 4. 정리

이번 포스트는 제네릭 컴파일러 경고 중 비검사 경고에 대해서 알아보았습니다.

비검사 경고는 ClassCastException이 발생할 가능성이 있을 때 나오는 경고로

코드를 타입 안전하게 만들고 싶다면 비검사 경고를 최대한 제거해야 합니다.

만약, 경고를 제거할 수 없지만 타입이 안전하다고 보장되면 

가능한 좁은 범위에 SuppressWarnings 어노테이션을 사용합시다.

## 5. 참조

* [https://iwannafullstack.tistory.com/2](https://iwannafullstack.tistory.com/2)