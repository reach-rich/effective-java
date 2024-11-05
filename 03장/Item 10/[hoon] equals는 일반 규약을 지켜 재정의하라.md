## 1. 들어가기

equals 메서드는 객체가 동일한지 비교하는 메서드입니다.

각 클래스들은 Object 클래스의 equals 메서드를 재정의해 사용하곤 합니다.

하지만, equals 메서드는 곳곳에 함정이 도사리고 있어 자칫하면 끔찍한 결과를 초래하곤 합니다.

어떤 경우, 재정의를 하지 말아야 할까요?

## 2. 재정의를 하지 말아야하는 상황

재정의를 하지 말아야하는 상황 4가지를 알아보겠습니다.

* 각 인스턴스가 본질적으로 고유하다.

Integer나 String과 같이 값을 표현하는 클래스가 아닌
    
Thread처럼 각 인스턴스가 본질적으로 고유한 클래스는 재정의 하지 말아야 합니다.

* 인스턴스의 '논리적 동치성'을 검사할 일이 없다.

String이나 java.util.regex.Pattern의 equals 메서드처럼 논리적 동치성을 검사하는 경우를 제외한

나머지 경우는 Object의 equals 메서드로 해결됩니다.

* 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.

상위 클래스에서 재정의한 equals 메서드가 하위 클래스에도 딱 들어맞는다면

굳이 재정의할 필요 없겠죠?

* 클래스가 private이거나 package-private(default)이고 equals 메서드를 호출할 일이 없다.

보통 equals 메서드는 package 외부에서 사용하기 때문에
    
클래스가 private이거나 package-private이면 equals 메서드를 호출할 일이 없습니다.

만약, equals가 실수로라도 호출되는 것을 막고 싶은 경우, 아래와 같이 예외처리하면 됩니다.

```java
    @Override
    public boolean equals(Object o) {
        throw new AssertionError();
    }
```

## 3. 재정의를 해야하는 상황

그렇다면 equals를 재정의해야하는 상황은 언제일까요?

* 객체 식별성(두 객체가 물리적으로 동일한가)이 아닌 논리적 동치성을 확인해야한다.

주로 Integer, String과 같은 값 클래스들이 여기에 속합니다.

반면, 상위 클래스의 equals 메서드가 이미 논리적 동치성을 비교하도록 재정의 되었거나

값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스라면

다시 재정의할 필요가 없습니다.

## 4. equals 메서드를 재정의하기 위한 규약

equals 메서드를 재정의할 때는 반드시 일반 규약을 따라야 합니다.

5가지 일반 규약을 알아봅시다.

* 반사성 (Reflextivity)

모든 객체는 자기 자신과 같아야 합니다.
``` x.equals(x) == true ```

* 대칭성 (Symmetry)

두 객체는 서로에 대한 동치 여부에 똑같이 답해야합니다.
``` x.equals(y) == true ```
``` y.equals(x) == true ```

* 추이성 (Transitivity)

첫 번째 객체와 두 번째 객체가 같고, 두 번재 객체와 세 번째 객체가 같다면

첫 번째 객체와 세 번째 객체도 같아야 합니다.

``` x.equals(y) == true ```
``` y.equals(z) == true ```
``` x.equals(z) == true ```

* 일관성 (Consistency)

두 객체가 같다면 앞으로도 영원히 같아야 합니다.
``` x.equals(y) == true ```
``` x.equals(y) == true (5초 후) ```

* null-아님

모든 객체가 null과 같지 않아야 합니다.
``` x.equals(null) == false ```

## 5. equals 메서드 구현 방법

앞서 알아보았던 equals 메서드 재정의 규약을 바탕으로

어떻게 equals 메서드를 구현할 수 있는지 단계별로 알아보면서

예시를 직접 만들어 보겠습니다.

1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.

    자기 자신이면 true를 반환합니다.

    이는 단순 성능 최적화용으로 비교 작업이 복잡한 상황일 때 빛을 발할 것입니다.

    ```java
        if(o == this) {
            return true;
        }
    ```

2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.

    입력이 올바르지 않다면 false를 반환합니다.

    이 때 올바른 타입은 equals가 정의된 클래스인 것이 보통이지만,

    가끔 그 클래스가 구현한 특정 인터페이스가 될 수 있습니다.

    ```java
        if(!(o instanceof Book)) {
            return false;
        }
    ```

3. 입력을 올바른 타입으로 형변환한다.

    앞서 2번에서 instanceof를 통해 타입 검사를 했기 때문에 이 단계는 100% 성공합니다.

    ```java
        Book book = (Book)o;
    ```

4. 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다.

    모든 필드가 일치하면 true, 하나라도 다르면 false를 반환합니다.

    2번에서 인터페이스를 사용했다면 입력의 필드 값을 가져올 때도
    
    그 인터페이스의 메서드를 사용해야 합니다.

    ```java
        return book.name.equals(name) && book.writer.equals(writer);
    ```

아래 코드는 4가지를 종합해서 만든 예시 코드입니다.

```java
    @Override
    public boolean equals(Object o) {
        if(o == this)
            return true;

        if(!(o instanceof Book))
            return false;

        Book book = (Book)o;
        return book.name.equals(name) && book.writer.equals(writer);
    }
```

## 6. 주의 사항

equals 메서드를 재정의할 때 주의 사항을 알아보겠습니다.

1. equals를 재정의할 땐 hashcode도 반드시 재정의하자.

    hashcode를 재정의하지 않는다면 hashcode의 일반 규약을 어기게 되어 해당 클래스의 인스턴스를
    
    hashMap이나 hashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으킵니다.

2. 너무 복잡하게 해결하려 들지 말자.

    필드들의 동치성만 검사해도 equals 규약을 어렵지 않게 지킬 수 있습니다.

    오히려 더 공격적으로 파고들다가 문제를 일으키곤 합니다.

3. Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자.

    많은 프로그래머들이 매개변수를 Object가 아닌 다른 클래스로 두곤 합니다.

    이렇게 되면 Object.equals를 재정의한게 아닌 다중정의를 한 것이 됩니다.

## 7. 정리

이번 포스트에서는 Object 클래스의 equals 메서드를

재정의하지 말아야할 상황과 해야할 상황을 알아보면서

재정의 해야할 경우, 일반 규약을 지킨 구현 방법에 대해서 알아보았습니다.

하지만, 꼭 필요한 경우가 아니면 equals 메서드를 재정의하지 않는 것이 좋습니다.

많은 경우에 Object의 equals 메서드가 원하는 비교를 정확하게 수행해주기 때문이죠.

그래도 어쩔 수 없이 재정의가 필요한 경우에는 앞서 알아보았던 5가지 규약을 지키며 재정의해야 합니다.