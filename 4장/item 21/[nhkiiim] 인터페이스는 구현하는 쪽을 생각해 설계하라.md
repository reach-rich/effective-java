# 인터페이스는 구현하는 쪽을 생각해 설계하라

### 1. 인테페이스에서 디폴트 메서드
- 자바 8 전에는 구현체를 깨뜨리지 않고 인터페이스 메서드를 추가할 방법이 없었다.
- 인터페이스에 메서드를 추가하면 컴파일 오류!
- 자바 8에서 기존 인터페이스에 메서드를 추가할 수 있도록 디폴트 메서드를 소개했지만 `->` 아직 위험!!

<br>

- 디폴트 메서드를 선언 `->` 인터페이스 구현 후 재정의 하지 않은 모든 클래스에서 디폴트 메서드 사용
- 디폴트 메서드는 구현 클래스들 아무것도 모르는데 무작정 삽입될 뿐!!!

#
### 2. 자바 8에서 추가된 디폴트 메서드
- 핵심 컬렉션 인터페이스들ㄹ에 다수의 디폴트 메서드가 추가됨 (주로 람다를 활용하기 위해)
- 자바 라이브러리의 디폴트 메서드는 품질이 높고 범용적
- BUT! 생각할 수 있는 모든 상황에서 불변식을 해치지 않는 디폴트 메서드를 작성하긴 어려움

<br>

- __자바 8의 Collection 인터페이에 추가된 removeId 메서드__
  - 주어진 불리언 함수가 true를 반환하는 모든 원소를 제거
  - 잘 짜여진 코드이지만, 현존하는 모든 Collection 구현체와 잘 어울리는 것은 아니다! 
    - ex) SynchronizedCollection : 한 스레드에서 removeIf를 호출해버리면 동기화 실패!
  - 자바에서도 이런 문제를 예방하기 위해 디폴트메서드를 재정의하고, 호출하기 전 필요한 작업을 수행하게 하는 작업들을 적용

```java
default boolean removeIf(Predicate<? super E> filter){
  Objects.requireNonNull(filter);
  boolean result = false;
  
  for(Iterator<E> it = iterator(); it.hasNext();){
    if(filter.test(it.next())){
      it.remove();
      result = true;
    }
  }
  
  return result;
}
```
  * Predicate : https://yeonyeon.tistory.com/200
  * SynchronizedCollection : https://docs.microsoft.com/ko-kr/dotnet/api/system.collections.generic.synchronizedcollection-1?view=dotnet-plat-ext-6.0

#
### 3. 주의할 점
- 디폴트 메서드는 기존 구현체의 런타임오류를 부를 수도 있다! (흔한 일은 아니지만 일어날 수도 있음)
- 자바 8이 컬렌션에 많은 디폴트 메서드를 추가했고 그 결과 기존의 코드들이 영향을 받음
- 기존 인터페이스에 디폴트 메서드를 추가하는 일은 꼭 필요한 일이 아니면 피해야함

- 새로운 인터페이스를 만들 때 유용한 수단으로 사용하는 건 좋다!
- 하지만! 인터페이스로부터 기존 메서드를 제거하거나 기존 메서드의 시그니처를 수정하는 용도로는 절대 안된다

<br>

`핵심` __인터페이스를 설계할 때는 세심한 주의를 기울이자__
  - 테스트도 충분히 하자! 인스턴스를 여러개 다양한 용도로 만들어보자!





