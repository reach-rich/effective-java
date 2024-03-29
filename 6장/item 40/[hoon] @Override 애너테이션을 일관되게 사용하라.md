## 1. 들어가기

프로그래머들이 사용하는 애너테이션 중 가장 많이 사용하는 것은 아마 @Override일 것입니다.

@Override는 메서드 선언에만 붙일 수 있으며,

이 애너테이션이 달린 메서드는 상위 타입의 메서드를 정의했음을 뜻합니다.

하지만 @Override를 빼먹어도 문제없이 실행 되기 때문에 종종 빼먹기도 하는데

그럴 경우 어떤 문제가 발생할 수 있는지 알아보겠습니다.

## 2. @Override 누락으로 발생할 수 있는 문제점

```java
  public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
      this.first = first;
      this.second = second;
    }

    public boolean equals(Bigram b) {
      return b.first == first && b.second == second;
    }

    public int hashCode() {
      return 31 * first + second;
    }

    public static void main(String[] args) {
      Set<Bigram> s = new HashSet<>();
      for(int i=0; i<10; i++) {
        for(char ch='a'; ch <='z'; ch++) {
          s.add(new Bigram(ch, ch));
        }
      }
      System.out.println(s.size());
    }
  }
```

위의 예시는 똑같은 소문자 2개로 구성된 바이그램 26개를 10번 반복해 집합에 추가한 다음,

그 집합의 크기를 구한 예시입니다.

언뜻 보기에는 Set은 중복을 허용하지 않기 때문에 26이 출력될 것 같지만, 실제로는 260이 출력됩니다.

왜 그럴까요? 🤔

예시를 다시 보면 Bigram 작성자는 equals와 hashCode 메서드를 재정의하려 한듯 보입니다.

하지만, equals 메서드의 매개변수를 자세히 보면 `Bigram` 타입인 것을 볼 수 있습니다.

즉, Bigram 작성자는 equals 메서드를 재정의(Override)하려 했지만,

실은 다중정의(Overriding)를 한 것입니다.

그렇기에 기본 equals 메서드인 Object의 equals 메서드로 Set이 동작하게 되는데

Object의 equals 메서드는 객체 식별성만을 확인하기 때문에 결국 260을 출력하게 된 것입니다.

```java
  @Override
  public boolean equals(Bigram b) {
    return b.first == first && b.second == second;
  }
```

만약, Bigram 작성자가 위와 같이 재정의한 메서드에 @Override 하나만 붙였다면

컴파일 시간에 오류를 알 수 있었을 것이고, 아래와 같이 올바르게 수정할 수 있었을 것입니다.

```java
  @Override
  public boolean equals(Object o) {
    if(!(o instanceof Bigram)) {
      return false;
    }

    Bigram b = (Bigram) o;
    return b.first == first && b.second == second;
  }
```

## 3. @Override 애너테이션을 붙이지 않아도 되는 한 가지 예외

앞서 재정의하는 메서드는 모두 @Override 애너테이션을 붙이도록 권장했습니다.

하지만, 하나의 예외가 있습니다.

바로, 구체 클래스에서 상위 클래스의 추상 메서드를 재정의할 때인데요.

구체 클래스인데 아직 구현하지 않은 추상 메서드가 남아있다면 컴파일러가 알려주기 때문입니다.

하지만, @Override 애너테이션을 붙인다고 해서 나쁠 것은 없습니다.

오히려 여러 IDE들은 @Override 애너테이션을 사용하도록 부추깁니다.

@Override를 일관되게 사용하면 실수로 재정의했을 때 컴파일러가 경고해주기 때문입니다.

## 4. 정리

이번 포스트는 @Override 애너테이션에 대해 알아보았습니다.

@Override 애너테이션은 메서드 재정의를 하면서 실수할 때, 컴파일러가 바로 알려주는 장점이 있습니다.

한 가지 예외로 구체 클래스에서 상위 클래스의 추상 메서드를 재정의할 때가 있는데

@Override 애너테이션을 붙인다고 해서 나쁠 것은 없으므로 @Override 애너테이션 사용을 지향합시다.