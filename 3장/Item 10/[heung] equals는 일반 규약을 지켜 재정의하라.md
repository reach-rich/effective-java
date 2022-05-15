Effective Java의 열 번째 아이템 "equals는 일반 규약을 지켜 재정의하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 0. 들어가며

equals 메서드는 재정의하기 쉬워 보이지만 곳곳에 함정이 도사리고 있어서 자칫하면 끔찍한 결과를 초래합니다. 본문에서 equals 메서드의 재정의가 어떠한 상황에서 필요한지 그리고 올바른 구현 방법과 주의사항을 알아보겠습니다. 

<br>

## 1. 재정의가 불필요한 경우

equals로 인해 발생하는 문제를 회피하는 가장 쉬운 길은 아예 재정의하지 않는 것입니다. 그냥 두면 그 클래스의 인스턴스는 오직 자기 자신과만 같게 됩니다.

다음은 equals 메서드의 재정의가 필요하지 않는 경우입니다. 

- **각 인스턴스가 본질적으로 고유하다.**

  값을 표현하는 것이 아니라 동작하는 개체를 표현하는 클래스가 여기 해당합니다. 예를 들어 Thread 클래스가 있습니다. 

- **인스턴스의 '논리적 동치성(logical equality)'을 검사할 일이 없다.**

  논리적 동치성이란 두 인스턴스가 논리적으로 같음을 의미합니다. 예를 들어 java.util.regex.Parrern의 두 인스턴스가 같은 정규표현식을 나타내는 경우가 있습니다. 이러한 논리적 동치성을 검사할 일이 없다면 equals 재정의는 필요하지 않습니다.

- **상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.**

  예를 들어 대부분의 Set 구현체는 AbstractSet이 구현한 equals를 상속받아쓰고, List 구현체들은 AbstractList로부터, Map 구현체들은 AbstractMap으로부터 상속받아 그대로 씁니다.

- **클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.**

  이 경우에는 당연히 재정의가 필요하지 않습니다.

위에서 열거한 상황 중 하나에 해당한다면 재정의하지 않는 것이 '최선'입니다.

<br>

## 2. 재정의가 필요한 경우

equals 메서드의 재정의가 필요한 경우는 두 객체가 물리적으로 같은지가 아니라 논리적으로 같은지, 즉 논리적 동치성을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때입니다.

주로 Integer와 String처럼 값을 표현하는 클래스들을 equals로 비교하는 프로그래머는 객체가 같은지 보다는 값이 같은지를 알고 싶어 할 것입니다. equals가 논리적 동치성을 확인하도록 정의해두면, 그 인스턴스는 값을 비교하길 원하는 프로그래머의 기대에 부응함을 물론 Map의 키와 Set의 원소로 사용할 수 있게 됩니다.

<br>

## 3. 규약

equals 메서드를 재정의할 때는 반드시 일반 규약을 따라야 합니다. 다음은 Object 명세에 적힌 규약입니다.

- **반사성**

  null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true다. 단순히 말하면 객체는 자기 자신과 같이야 한다는 뜻입니다.

- **대칭성**

  null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)가 true면 y.equals(x)도 true다. 두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다는 뜻입니다.

- **추이성**

  null이 아닌 모든 참조 값 x, y, z에 대해, x.equals(y)가 true이고 y.equals(z)도 true이면 x.equals(z)도 true다. 첫 번째 객체와 두 번째 객체가 같고, 두 번째 객체와 세 번째 객체가 같다면, 첫 번째 객체와 세 번째 객체도 같아야 한다는 뜻입니다.

- **일관성**

  null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다. 두 객체가 같다면 어느 하나 혹은 두 객체 모두가 수정되지 않는 한 앞으로도 영원히 같아야 한다는 뜻입니다. 클래스가 불변이든 가변이든 equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안 됩니다.

- **not-null**

  null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다. 모든 객체가 null과 같지 않아야 한다는 뜻입니다. 

이 규약을 어기면 프로그램이 이상하게 동작하거나 종료될 것이고, 원인이 되는 코드를 찾기도 굉장히 어려울 것입니다. 컬력션 클래스들을 포함해 수많은 클래스는 전달받은 객체가 equals 규약을 지킨다고 가정하고 동작합니다.

사실 반사성과 not-null은 일반적으로 만족하게 될 것입니다. 하지만 대칭성, 추이성, 일관성은 자칫하면 어길 수 있기 때문에 주의가 필요합니다.

<br>

## 4. 올바른 구현 방법

1. **== 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.**

   자기 자신이면 true를 반환합니다. 이는 단순한 성능 최적화용으로, 비교 작업이 복잡한 상황일 때 값어치를 할 것입니다.

2. **instanceof 연산자로 입력이 올바른 타입인지 확인한다.**

   그렇지 않다면 false를 반환합니다. 이때의 올바른 타입은 equals가 정의된 클래스인 것이 보통이지만, 가끔은 그 클래스가 구현한 특정 인터페이스가 될 수도 있습니다. 

   어떤 인터페이스는 자신을 구현한 클래스끼리도 비교할 수 있도록 equals 규약을 수정하기도 합니다. 이런 인터페이스를 구현한 클래스라면 equals에서 해당 인터페이스를 사용해야 합니다. Set, List, Map, Map.Entry 등의 컬렉션 인터페이스들이 여기 해당합니다.

3. **입력을 올바른 타입으로 형변환한다.**

   앞서 2번에서 instanceof 검사를 했으면 이 단계는 무조건 성공합니다.

4. **입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다.**

   모든 필드가 일치하면 true를, 하나라도 다르면 false를 반환합니다. 2단계에서 인터페이스를 사용했다면 입력의 필드 값을 가져올 때도 그 인터페이스 메서드를 사용해야 합니다. 타입이 클래스라면 해당 필드에 직접 접근할 수도 있습니다.

다음은 위의 구현 방법을 따라 작성한 PhoneNumber 클래스용 equals 메서드입니다.

```java
public final class PhoneNumber {
    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = rangeCheck(areaCode, 999, "지역코드");
        this.prefix   = rangeCheck(prefix,   999, "프리픽스");
        this.lineNum  = rangeCheck(lineNum, 9999, "가입자 번호");
    }

    private static short rangeCheck(int val, int max, String arg) {
        if (val < 0 || val > max)
            throw new IllegalArgumentException(arg + ": " + val);
        return (short) val;
    }

    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof PhoneNumber))
            return false;
        PhoneNumber pn = (PhoneNumber)o;
        return pn.lineNum == lineNum && pn.prefix == prefix
                && pn.areaCode == areaCode;
    }

    ...
}
```

<br>

## 5. 주의사항

- **equals를 재정의할 땐 hashCode도 반드시 재정의하자.**

- **너무 복잡하게 해결하려 들지 말자.**

  필드들의 동치성만 검사해도 equals 규약을 어렵지 않게 지킬 수 있습니다. 오히려 너무 공격적으로 파고들다가 문제를 일으키기도 합니다. 일반적으로 별칭(alias)은 비교하지 않는 것이 좋습니다. 예를 들어 File 클래스라면, 심볼릭 링크를 비교해 같은 파일을 가리키는지를 확인하려 들면 안 됩니다.

- **Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자.**

  `public boolean equals(MyClass o) { ... }`와 같은 메서드는 Object.equals를 재정의 한 것이 아닙니다. 입력 타입이 Object가 아니므로 재정의가 아니라 다중정의한 것 입니다. 기본 equals를 그대로 둔 채로 추가한 것일지라도, 이처럼 타입을 구체적으로 명시한 equals는 오히려 해가 됩니다. 이 메서드는 하위 클래스에서 @Override이 긍정 오류를 내게 하고 보안 측면에서도 잘못된 정보를 줍니다. 

<br>

## 6. 테스트

equals를 작성하고 테스트하는 일은 지루하고 이를 테스트하는 코드도 항상 뻔합니다. 이러한 작업을 대신해 줄 오픈소스가 있으니, 바로 구글이 만든 **AutoValue** 프레임워크입니다. 클래스에 어노테이션 하나만 추가하면 AutoValue가 이 메서드들을 알아서 작성해 주며, 직접 작성하는 것과 근본적으로 똑같은 코드를 만들어 줄 것입니다.

<br>

## 7. 핵심 정리

꼭 필요한 경우가 아니면 equals를 재정의하지 말자. 많은 경우에 Object의 equals가 여러분이 원하는 비교를 정확히 수행해 준다. 재정의해야 할 때는 그 클래스의 핵심 필드 모두를 빠짐없이, 다섯 가지 규약을 확실히 지켜가며 비교해야 한다.

<br>

## 8. Related Posts

- [인스턴스 통제 클래스 (Item 1)](https://heung27.github.io/posts/effective-java-item-1-%EC%83%9D%EC%84%B1%EC%9E%90-%EB%8C%80%EC%8B%A0-%EC%A0%95%EC%A0%81-%ED%8C%A9%ED%84%B0%EB%A6%AC-%EB%A9%94%EC%84%9C%EB%93%9C%EB%A5%BC-%EA%B3%A0%EB%A0%A4%ED%95%98%EB%9D%BC/)
- 불변 클래스 (Item 17)
- hashCode (Item 11)
- 다중정의 (Item 52)
- @Override (Item 40)
