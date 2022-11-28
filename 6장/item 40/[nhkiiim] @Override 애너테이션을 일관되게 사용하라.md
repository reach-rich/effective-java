# @Override 애너테이션을 일관되게 사용하라

자바가 기본으로 제공하는 애너테이션 중 보통 프로그래머에게 가장 중요한 @Override!!
@Override는 메서드 선언에만 달 수 있으며, 상위 타입 매서드를 재정의했음을 의미한다.

이 애너테이션을 일관되게 사용하면 여러 악명 높은 버그들을 예방해준다..!

__영어 알파벳 2개로 구성된 문자열을 표현하는 클래스__
```java
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) 
        this.first = first;
        this.second = second;
    }

    public boolean equals(Bigram bigram) {
        return bigram.first == this.first && bigram.second == this.second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++) {
            for (char ch = 'a'; ch <= 'z'; ch++) {
                s.add(new Bigram(ch, ch));
            }
        }

        System.out.println(s.size()); // 결과 260
    }
}

```
26개의 set을 만들지만 260이라는 사이즈가 나온다. (같은 set이 계속 중복해서 들어감)

-> equals 메서드를 재정의 할 때 hashCode도 함께 재정의 해야하는데 그러지 않았기 때문!

<br>

위와 같은 코드는 equals를 재정의 한 것이 아니라 오버로딩 해버린 것이 된다!
이 오류를 컴파일러가 찾아내기 위해서는 @Override 애너테이션을 달아줘야한다.

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

<br>

__그러니 상위 클래스의 메서드를 재정의 하는 모든 메서드에는 @Override 애너테이션을 달자!__

<br>

예외가 한가지 존재하긴 한다!

바로 구체 클래스 (모든 연산에 대한 구현을 가지고 있는 클래스) 에서 상위 클래스의 추상 메서드를 정의할 때 이다!
구체 클래스인데 아직 구현하지 않은 추상 메서드가 남아있다면 컴파일러가 바로 알려주기 때문에 붙이지 않아도 되지만
일관성을 주기 위해 붙인다면? 그것도 상관 없다! (대부분의 IDE는 @Override를 일괄적으로 붙여준다)

<br>

@Override는 클래스 뿐만 아니라 인터페이스 메서드를 재정의 할 때도 사용할 수 있다.
디폴트 메서드가 나오면서 디폴트 메서드와 구분하기 위한 좋은 방법이 되기도 한다!
