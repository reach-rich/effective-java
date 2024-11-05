Effective Java의  40번째 아이템 "@Override 애너테이션을 일관되게 사용하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 1. @Override

@Override는 메서드 선언에만 달 수 있으며, 이 애너테이션이 달렸다는 것은 상위 타입의 메서드를 재정의했음을 뜻한다. 이 애너테이션을 일관되게 사용하면 여러 가지 악명 높은 버그들을 예방해준다. 

다음 Bigram 프로그램을 살펴보자. 이 클래스는 바이그램, 즉 여기서는 영어 알파벳 2개로 구성된 문자열을 표현한다.

```java
public class Bigram {
  private final char first;
  private final char second;
  
  public Bigram(char first, char second) {
    this.first = first;
    this.second = second;
  }
  
  public boolean equals(Bigram b) {
    return b.first = first && b.second = second;
  }
  
  public int hashCode() {
    return 31 * first + second;
  }
  
  public static void main(String[] args) {
    Set<Bigram> s = new HashSet<>();
    for (int i = 0; i < 10; i++) {
      for (char ch = 'a'; ch <= 'z'; ch++)
        s.add(new Bigram(ch, ch));
      System.out.println(s.size());
    }
  }
}
```

똑같은 소문자 2개로 구성된 바이그램 26개를 10번 반복해 집합에 추가한 다음, 그 집합의 크기를 출력한다. Set은 중복을 허용하지 않으니 26이 출력될 것 같지만, 실제로는 260이 출력된다.

원인은 equals 메서드를 '**재정의**'한 게 아니라 '**다중정의**'해 버렸기 때문이다. Object의 equals를 재정의하려면 매개변수 타입을 Object로 해야만 하는데, 그렇게 하지 않아 Object에서 상속한 equals와는 별개인 equals를 새로 정의한 꼴이 되었다. 따라서 같은 소문자를 소유한 바이그램 10개 각각이 서로 다른 객체로 인식되고, 결국 260을 출력한 것이다.

다행히 이 오류는 컴파일러가 찾아낼 수 있지만, 그러려면 Object.equals를 재정의한다는 의도를 명시해야 한다.

```java
@Override
public boolean equals(Bigram b) {
  return b.first == first && b.second == second;
}
```

이처럼 @Override 애너테이션을 달고 다시 컴파일하면 컴파일 오류가 발생하고, 잘못된 부분을 명확히 알려주므로 곧장 올바르게 수정할 수 있다.

그러니 **상위 클래스의 메서드를 재정의하려는 모든 메서드에 @Override 애너테이션을 달자.** 한가지 예외는 있는데, 구체 클래스에서 추상 메서드를 재정의할 때는 굳이 @Override를 달지 않아도 된다. 물론 재정의 메서드 모두에 @Override를 일괄로 붙여두는 게 좋아 보인다면 그래도 상관없다.

<br>

## 2. 핵심 정리

* 재정의한 모든 메서드에 @Override 애너테이션을 의식적으로 달면 여러분이 실수했을 때 컴파일러가 바로 알려줄 것이다.
* 예외는 한 가지뿐이다. 구체 클래스에서 상위 클래스의 추상 메서드를 재정의한 경우엔 이 애너테이션을 달지 않아도 된다(단다고 해서 해로울 것도 없다).

<br>

## 3. Related Posts

* [equals (Item 10)](https://heung27.github.io/posts/item-10-equals%EB%8A%94-%EC%9D%BC%EB%B0%98-%EA%B7%9C%EC%95%BD%EC%9D%84-%EC%A7%80%EC%BC%9C-%EC%9E%AC%EC%A0%95%EC%9D%98%ED%95%98%EB%9D%BC/)
* [hashCode (Item 11)](https://heung27.github.io/posts/item-11-equals%EB%A5%BC-%EC%9E%AC%EC%A0%95%EC%9D%98%ED%95%98%EB%A0%A4%EA%B1%B0%EB%93%A0-hashcode%EB%8F%84-%EC%9E%AC%EC%A0%95%EC%9D%98%ED%95%98%EB%9D%BC/)
* 오버로딩 (Item 52)
