Effective Java의  42번째 아이템 "익명 클래스보다는 람다를 사용하라"를 읽고 정리한 내용을 포스팅합니다.

## 1. 익명 클래스

예전에는 자바에서 함수 타입을 표현할 때 추상 메서드를 하나만 담은 인터페이스(드물게는 추상 클래스)를 사용했다. 이런 인터페이스의 인스턴스를 함수 객체(function object)라고 하여, 특정 함수나 동작을 나타내는 데 썼다.

1997년 JDK 1.1이 등장하면서 함수 객체를 만드는 주요 수단은 **익명 클래스**가 되었다. 다음은 문자열을 길이순으로 정렬하기 위한 비교 함수로 익명 클래스를 사용한 예다.

```java
Collections.sort(words, new Comparator<String>() {
  public int compare(String s1, String s2) {
    return Integer.compare(s1.length(), s2.length());
  }
});
```

이 코드에서 Comparator 인터페이스가 정렬을 담당하는 추상 전략을 뜻하며, 문자열을 정렬하는 구체적인 전략을 익명 클래스로 구현했다. 하지만 익명 클래스 방식은 코드가 너무 길기 때문에 자바는 함수형 프로그래밍에 적합하지 않았다.

<br>

## 2. 람다

자바 8에 와서 추상 메서드 하나짜리 인터페이스는 특별한 의미를 인정받아 특별한 대우를 받게 되었다. 지금은 함수형 인터페이스라 부르는 이 인터페이스들의 인스턴스를 **람다식**을 사용해 만들 수 있게 된 것이다.

람다는 함수나 익명 클래스와 개념은 비슷하지만 코드는 훨씬 간결하다. 다음은 익명 클래스를 사용한 앞의 코드를 람다 방식으로 바꾼 모습이다.

```java
Collections.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length()));
```

여기서 람다, 매개변수(s1, s2), 반환값의 타입은 각각 (Comparator\<String>), String, int지만 코드에서는 언급이 없다. 우리 대신 컴파일러가 문맥을 살펴 타입을 추론해준 것이다. 상황에 따라 컴파일러가 타입을 결정하지 못할 수도 있는데, 그럴 때는 프로그래머가 직접 명시해야 한다.

타입 추론 규칙은 너무 복잡하고, 잘 알지 못한다 해도 상관없다. **타입을 명시해야 코드가 더 명확할 때만 제외하고는, 람다의 모든 매개변수 타입은 생략하자.** 그런 다음 컴파일러가 "타입을 알 수 없다"는 오류를 낼 때만 해당 타입을 명시하면 된다. 

람다가 꼭 좋은 것만은 아니다. 메서드나 클래스와 달리, 람다는 이름이 없고 문서화도 못 한다. 따라서 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 람다를 쓰지 말아야 한다. 

<br>

## 3. 익명 클래스 vs 람다

람다의 시대가 열리면서 익명 클래스는 설 자리가 크게 좁아진 게 사실이다. 하지만 람다로 대체할 수 없는 곳이 있다. 

* **람다는 함수형 인터페이스에서만 쓰인다.** 
  * 추상 클래스의 인스턴스를 만들 때 람다를 쓸 수 없으니, 익명 클래스를 써야 한다. 
  * 추상 메서드가 여러 개인 인터페이스의 인스턴스를 만들 대도 익명 클래스를 쓸 써야 한다.

* **람다는 자신을 참조할 수 없다.** 
  * 람다에서의 this 키워드는 바깥 인스턴스를 가리킨다. 반면 익명 클래스에서의 this는 익명 클래스의 인스턴스 자신을 가리킨다. 그래서 함수 객체가 자기 자신을 참조해야 한다면 반드시 익명 클래스를 써야 한다. 
* **람다도 익명 클래스처럼 직렬화 형태가 구현별로 다를 수 있다.**
  * 람다를 직렬화하는 일은 극히 삼가야 한다(익명 클래스도 마찬가지다). 직렬화해야만 하는 함수 객체가 있다면 private 정적 중첩 클래스의 인스턴스를 사용하자.

<br>

## 3. 핵심 정리

* 자바가 8로 판올림되면서 작은 함수 객체를 구현하는 데 적합한 람다가 도입되었다.
* 익명 클래스는 타입의 인스턴스를 만들 때만 사용하라.
* 람다는 작은 함수 객체를 아주 쉽게 표현할 수 있어 함수형 프로그래밍의 지평을 열었다.

<br>

## 4. Related Posts

* 익명 클래스 (Item 24)
