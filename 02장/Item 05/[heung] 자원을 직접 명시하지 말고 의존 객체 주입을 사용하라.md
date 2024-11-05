Effective Java의 다섯 번째 아이템 "자원을 직접 명시하지 말고 의존 객체 주입을 사용하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 들어가며 

먼저 본문을 시작하기 전에 타이틀의 의미를 살펴보겠습니다. "**자원을 직접 명시한다**"가 무엇을 의미하는 걸까요? 코드를 보면 이해가 쉽습니다.

```java
public class SpellChecker {

    private Dictionary dictionary = new KorDictionary();

    ...
}
```

SpellChecker 클래스는 Dictionary 타입의 필드를 가지고 있습니다. 그리고 해당 필드(dictionary)는 선언과 동시에 초기화되고 있죠. 바로 이것이 자원을 직접 명시한다는 것입니다. 말 그대로 자원(dictionary)이 KorDictionary 클래스의 인스턴스임을 직접 명시하는 것입니다.

<br>

## 자원을 직접 명시한 예

본격적으로 예제를 통해 자세히 알아보겠습니다. 아래는 위에서 언급했던 Dictionary 인터페이스와 그것을 구현(implement)한 KorDictionary, EngDictionary 클래스의 전체 코드입니다.

```java
public interface Dictionary {

    boolean contains(String word);

    List<String> closeWordsTo(String typo);
}
```

```java
public class KorDictionary implements Dictionary {

    @Override
    public boolean contains(String word) {
        // TODO 단어가 한국어 사전에 포함되어 있는지 검사하는 로직
        return false;
    }

    @Override
    public List<String> closeWordsTo(String typo) {
        // TODO 오타와 비슷한 한국어 단어를 찾는 로직
        return null;
    }
}
```

```java
public class EngDictionary implements Dictionary {

    @Override
    public boolean contains(String word) {
        // TODO 단어가 영어 사전에 포함되어 있는지 검사하는 로직
        return false;
    }

    @Override
    public List<String> closeWordsTo(String typo) {
        // TODO 오타와 비슷한 영어 단어를 찾는 로직
        return null;
    }
}
```

예제를 간단히 하여 본문의 주제에만 집중하기 위해 메서드는 구현 코드는 주석으로 대체하였습니다. 

<br>

이제 자원을 명시하는 SpellChecker 클래스를 보겠습니다. 첫 번째는 정적 유틸리티 클래스, 두 번째는 싱글턴으로 구현한 모습입니다.

```java
// 정적 유틸리티 클래스
public class SpellChecker {

    private static final Dictionary dictionary = new KorDictionary();

    private SpellChecker() {}

    public static boolean isValid(String word) {
        return dictionary.contains(word);
    }

    public static List<String> suggestions(String typo) {
        return dictionary.closeWordsTo(typo);
    }
}
```

```java
// 싱글턴
public class SpellChecker {

    private final Dictionary dictionary = new KorDictionary();

    private SpellChecker() {}

    public static SpellChecker INSTANCE = new SpellChecker();

    public boolean isValid(String word) {
        return dictionary.contains(word);
    }

    public List<String> suggestions(String typo) {
        return dictionary.closeWordsTo(typo);
    }
}
```

정적 유틸리티 클래스와 싱글턴 모두 한국어 사전(KorDictionary)을 사용합니다. 그런데 우리는 영어 사전(EngDictionary)이 필요할 때도 있습니다. 위의 방식에서는 EngDictionary 타입의 필드를 추가하면 영어 사전을 사용할 수 있겠죠. 하지만 필요한 다른 종류의 사전이 늘어나면 어떻게 해야 할까요? 당연히 그 수만큼 필드를 추가하는 것은 비효율적일 것입니다. 

<br>

아주 간단히 생각해 보면 dictionary 필드에서 final 한정자를 제거하고, 다른 사전으로 교체하는 메서드를 추가할 수 있습니다.

```java
public void replaceDictionary(Dictionary dictionary) {
  this.dictionary = dictionary;
}
```

하지만 이 방식 역시 어색하고 오류를 내기 쉬우며 멀티스레드 환경에서는 쓸 수 없습니다. 결론적으로 **사용하는 자원에 따라 동작이 달라지는 클래스에게 정적 유틸리티 클래스나 싱글턴 방식은 적합하지 않습니다.**

<br>

## 의존 객체 주입

대신, "클래스(SpellChecker)가 여러 자원 인스턴스를 지원해야 하며 클라이언트가 원하는 자원(dictionary)을 사용해야 한다" 이 조건을 만족하는 간단한 패턴이 있습니다. 바로 **인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식**입니다. 이는 **의존 객체 주입의 한 형태**로, SpellChecker를 생성할 때 의존 객체인 Dictionary을 주입하는 것입니다.

```java
public class SpellChecker {

    private final Dictionary dictionary;

    public SpellChecker(Dictionary dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public boolean isValid(String word) {
        return dictionary.contains(word);
    }

    public List<String> suggestions(String typo) {
        return dictionary.closeWordsTo(typo);
    }
}
```

```java
SpellChecker korSpellChecker = new SpellChecker(new KorDictionary());
SpellChecker engSpellChecker = new SpellChecker(new EngDictionary());
```

예제에서는 dictionary라는 딱 하나의 자원만 사용하지만, 의존 객체 주입은 자원이 몇 개든 의존 관계가 어떻든 상관없이 잘 동작하고, 불변을 보장하여 여러 클라이언트가 의존 객체들을 안심하고 공유할 수 있도록 합니다. 또한 이전 포스팅에서 다루었던 생성자, 정적 팩터리, 빌더에도 똑같이 응용할 수 있습니다.

<br>

이 패턴의 변형으로, **생성자에 자원 팩터리를 넘겨주는 방식**이 있습니다. 여기서 팩터리란 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체를 말합니다. 즉, 팩터리 메서드 패턴을 구현한 것입니다.

Java 8에서 소개한 함수형 인터페이스 **Supplier\<T>**가 팩터리를 표현한 완벽한 예시입니다. Supplier\<T>를 입력으로 받는 메서드는 일반적으로 한정적 와일드카드 타입을 사용해 팩터리의 타입 매개변수를 제한해야 합니다. 이 방식을 사용해 클라이언트는 자신이 명시한 타입의 하위 타입이라면 무엇이든 생성할 수 있는 팩터리를 넘길 수 있습니다.

```java
public class SpellChecker {

    private final Dictionary dictionary;

    public SpellChecker(Supplier<? extends Dictionary> dictionarySupplier) {
        this.dictionary = dictionarySupplier.get();
    }

    public boolean isValid(String word) {
        return dictionary.contains(word);
    }

    public List<String> suggestions(String typo) {
        return dictionary.closeWordsTo(typo);
    }
}
```

 ```java
 SpellChecker korSpellChecker = new SpellChecker(KorDictionary::new);
 SpellChecker engSpellChecker = new SpellChecker(EngDictionary::new);
 ```

<br>

의존 객체 주입이 유연성과 테스트 용이성을 개선해 주기는 하지만, 의존성이 수 천 개나 되는 큰 프로젝트에서는 코드를 어지럽게 만들기도 합니다. 이런 어질러짐을 해소하기 위해서는 Dagger, Guice, Spring과 같은 의존 객체 주입 프레임워크를 사용할 수 있습니다.

<br>

## 핵심 정리

클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글턴과 정적 유틸리티 클래스를 사용하지 않는 것이 좋다. 이 자원들을 클래스가 직접 만들게 해서도 안 된다. 대신 필요한 자원을 (혹은 그 자원을 만들어주는 팩터리를) 생성자에 (혹은 정적 팩터리나 빌더에) 넘겨주자. 의존 객체 주입이라 하는 이 기법은 클래스의 유연성, 재사용성, 테스트 용이성을 기막히게 개선해 준다. 

<br>

## Related Post

- [정적 유틸리티 클래스 (Item 4)](https://heung27.github.io/posts/item-4-%EC%9D%B8%EC%8A%A4%ED%84%B4%EC%8A%A4%ED%99%94%EB%A5%BC-%EB%A7%89%EC%9C%BC%EB%A0%A4%EA%B1%B0%EB%93%A0-private-%EC%83%9D%EC%84%B1%EC%9E%90%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC/)
- [불변 (Item 17)]()
- [정적 팩터리 (Item 1)](https://heung27.github.io/posts/item-4-%EC%9D%B8%EC%8A%A4%ED%84%B4%EC%8A%A4%ED%99%94%EB%A5%BC-%EB%A7%89%EC%9C%BC%EB%A0%A4%EA%B1%B0%EB%93%A0-private-%EC%83%9D%EC%84%B1%EC%9E%90%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC/)
- [빌더 (Item 2)](https://heung27.github.io/posts/effective-java-item-2-%EC%83%9D%EC%84%B1%EC%9E%90%EC%97%90-%EB%A7%A4%EA%B0%9C%EB%B3%80%EC%88%98%EA%B0%80-%EB%A7%8E%EB%8B%A4%EB%A9%B4-%EB%B9%8C%EB%8D%94%EB%A5%BC-%EA%B3%A0%EB%A0%A4%ED%95%98%EB%9D%BC/)
