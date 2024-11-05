## 1. 들어가기

'정적 팩터리 메서드' 라는 용어를 들어본 적이 있나요?

용어가 다소 생소한 느낌이 있습니다.

지금부터 정적 팩터리 메서드가 무엇인지 알아봅시다.

## 2. Static Factory Method

### 2-1. 의미<br>

먼저 정적 팩터리 메서드라는 용어를 단어로 분리해볼까요?

> 정적(Static), 팩터리(Factory), 메서드(Method)

이 중 Factory라는 단어가 생소할 수 있습니다.

Factory의 사전적 의미는 '무언가를 만드는 공장' 입니다.

그럼 어떤 것을 만드는걸까요?

정적 팩터리 메서드의 Factory는 GoF 디자인 패턴 중 Factory 패턴에서 유래했는데,

WikiPedia에서 Factory 패턴에 대해서 찾아보면 다음과 같이 명시되어 있습니다.

> This is done by creating objects by calling a factory method—either specified in an interface and implemented by child classes

그리고 이를 해석하면 다음과 같습니다.

> 객체를 생성하는 인터페이스를 미리 정의하고, 자식 클래스에서 객체를 생성하는 패턴

여기서 Factory라는 것은 '객체 생성을 담당' 이라는 의미입니다.

자, 그럼 앞서 나눴던 정적, 팩터리, 메서드 이 세가지 단어를 합쳐볼까요?

> Static(인스턴스 생성 없이 사용가능하고) Factory(객체 생성을 담당하는) Method(멤버 함수)

드디어 정적 팩터리 메서드 용어의 의미를 알게 되었습니다.

그럼 예시를 통해 어떻게 사용하는지도 알아봅시다.

### 2-2. 예시<br>

우리가 흔히 아는 public 생성자를 통한 객체 생성 방법은 다음과 같습니다.

```
    Person person = new Person();
```

하지만, public 생성자를 통한 객체 생성 방법은

생성자의 매개변수만 보고 해당 객체가 어떤 용도로 사용되는지 파악해야하는 단점이 있습니다.

그래서 대안으로 '정적 팩터리 메서드' 라는 것을 사용할 수 있습니다.

java.lang 패키지에 속하는 Boolean 클래스의 valueOf 메서드를 통해 예시를 알아보겠습니다.

```
/* Boolean.class */

    public static final Boolean TRUE = new Boolean(true);
    public static final Boolean FALSE = new Boolean(false);

    public static Boolean valueOf(boolean b) {
        return (b ? TRUE : FALSE);
    }
```

Boolean 클래스의 valueOf 메서드는 

boolean 타입을 매개변수로 받아서 Boolean 객체를 생성해 반환하는 메서드입니다.

이쯤에서 이런 궁금증이 들 수 있습니다.

> 어떻게 사용하는지는 알겠는데 왜 '정적 팩터리 메서드' 를 통해 객체를 생성하나요?

한번 더 감싸면서 개발자가 더 많은 작업을 해줘야하는 것 같고...

아직까지는 어떤 이유로 객체를 '정적 팩토리 메서드' 를 통해 생성하는지 모르겠네요. 

장점과 단점을 통해 왜 권장하는지 알아봅시다.

### 2-3. 장점

<br>
 1. 이름을 가질 수 있다.
<hr>

정적 팩터리 메서드는 Boolean 클래스의 valueOf처럼 이름을 가질 수 있습니다.

이름을 가지게 되면 어떤 장점이 있을까요?

java.math 패키지에 속하는 BigInteger 클래스로 예시를 들어보겠습니다.

```
/* BigInteger.class */

    // 생성자
    public BigInteger(int bitLength, int certainty, Random rnd) {
        ...
    } 

    // 정적 팩터리 메서드
    public static BigInteger probablePrime(int bitLength, Random rnd) {
        ...
    }
```

위의 코드를 유지보수 하게 됐다고 가정해봅시다.

만약, 객체를 public 생성자를 통해 생성했다면

개발자가 어떤 의도로 객체를 생성했는지 매개변수만 보고 유추해야합니다.

심지어 유추를 하더라도 개발자가 만든 의도와 다를 확률이 높습니다.

반면, 객체를 정적 팩터리 메서드를 통해 생성했다면

probablePrime 이라는 메서드 이름만 보고도 

개발자가 어떤 의도로 객체를 생성했는지 한눈에 파악할 수 있습니다.

<br>
 2. 시그니처에 제약이 없다.
<hr>

public 생성자는 다음과 같이 동일한 이름을 가진 여러 개의 생성자를 만들 수 있습니다.

```
    public class Person {
        String name, gender;

        public Person() {
            ...
        }

        public Person(String name) {
            ...
        }

        public Person(String name, String gender) {
            ...
        }
    }
```

public 생성자는 여러 개의 생성자를 만들 수 있지만

다음과 같이 시그니처가 같은 경우에는 생성자를 만들지 못하는 문제점이 있습니다.

```
    public class Person {
        String name, gender;

        public Person(String name) {
            ...
        }

        public Person(String gender) { // 불가
            ...
        }
    }
```

반면, 정적 팩터리 메서드는 시그니처 제약이 없습니다.

예시를 통해 알아봅시다.

```
    public class Person {
        String name, gender;

        public static Person personWithName(String name) {
            Person person = new Person();
            person.name = name;

            return person;
        }

        public static person personWithGender(String gender) {
            Person person = new Person();
            person.gender = gender;

            return person;
        }
    }
```

예시를 보면 두 메서드의 매개변수 타입과 개수가 같지만, 

정적 팩터리 메서드를 통해 다른 객체를 생성할 수 있습니다.

뿐만 아니라 각 메서드 별로 이름이 다르기에 

어떤 용도로 만들었는지 한 눈에 알 수 있습니다.

<br>
 3. 호출할 때마다 새로운 객체를 생성하지 않아도 된다.
<hr>

앞서 정적 팩터리 메서드의 용어를 단어로 나누어 설명할 때,

static 키워드를 사용하면 객체를 생성하지 않아도 된다고 언급을 잠깐 했었습니다.

그럼 이런 궁금증이 들 수 있습니다.

> 객체를 생성하지 않으면 어떤 점이 좋은가요?

만약, 생성 비용이 큰 같은 객체가 자주 요청되는 상황이 있다고 생각해봅시다.

매번 public 생성자를 통한 방법은 생성 비용이 기하급수적으로 늘어나게 됩니다.

반면, 정적 팩터리 메서드는 객체를 캐싱해서 재활용하는 방법을 사용합니다.

그래서 객체를 매번 생성하는 것과 결과는 똑같지만 성능을 상당히 끌어올릴 수 있게 됩니다.

또한, 정적 팩터리 메서드는 인스턴스 통제(instance-controlled) 클래스입니다.

인스턴스 통제 클래스는 언제 어느 시점에 인스턴스를 살아있게 할지 통제할 수 있는 클래스입니다.

그래서 클래스를 싱글턴(Singleton), 인스턴스 불가로 만들 수 있고,

인스턴스가 하나 뿐임을 보장할 수 있습니다.

<br>
 4. 반환 타입의 하위 타입 객체를 반환할 수 있다.
<hr>

정적 팩터리 메서드는 다형성을 이용하기에 유연성이 있습니다.

예시를 통해 알아봅시다.

```
    public interface Person {
        static Person getMan() {
            return new Man();
        }

        static Person getWoman() {
            return new Woman();
        }
    }

    class Man implements Person {
        ...
    }

    class Woman implements Person {
        ...
    }
```

해당 코드는 Person 인터페이스와 이것을 구현하는 Man, Woman 클래스로 구성되어 있습니다.

getMan과 getWoman 메서드의 반환 타입은 Person이지만,

반환하고 있는 객체는 인터페이스의 하위 타입인 것을 알 수 있습니다.

그래서 이를 사용하면 인터페이스 기반의 프레임워크를 쉽게 만들 수 있습니다.

<br>
 5. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
<hr>

반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환해도 상관없습니다.

EnumSet 클래스의 noneOf 메서드를 예시로 들어보겠습니다.

```
/* EnumSet.class */

    public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
        Enum<?>[] universe = getUniverse(elementType);
        if (universe == null)
            throw new ClassCastException(elementType + " not an enum");

        if (universe.length <= 64)
            return new RegularEnumSet<>(elementType, universe);
        else
            return new JumboEnumSet<>(elementType, universe);
    }
```

우리가 주의깊게 봐야하는 곳은 6번째 줄입니다.

원소 수가 64개 이하면 long 변수로 관리하는 RegularEnumSet 인스턴스를,

원소 수가 64개 초과면 long 배열로 관리하는 JumboEnumSet 인스턴스를 반환합니다.

만약, RegularEnumSet을 다른 클래스로 수정하더라도

EnumSet의 하위 타입이기만 하면 문제없이 사용할 수 있습니다.

<br>
 6. 정적 팩터리 메서드를 작성하는 시점에는 반환할 클래스가 존재하지 않아도 된다.
<hr>

반환할 객체의 클래스가 존재하지 않아도 인터페이스나 클래스를 상속 받는 상황이라면,

언제든지 의존성을 주입 받아서 사용이 가능합니다.

```
    public class Man implements Person {
        public static List<Person> getFriend() {
            ...
        }
    }
```

이러한 유연함은 서비스 제공자 프레임워크를 만드는 근간이 됩니다.

### 2-4. 단점

<br>
 1. 정적 팩터리 메서드만 제공한다면 하위 클래스를 생성할 수 없다.
<hr> 

보통 정적 팩터리 메서드를 통해 객체를 생성한다면,

생성자의 접근 제한자를 private으로 설정해 외부에서의 객체 생성을 막습니다.

그러므로 해당 클래스를 상속하지 못하는 단점이 있습니다.

하지만, 상속보다는 컴포지션 사용과 클래스를 불변 타입으로 만드는 것을 권장하므로

이는 단점이 아닌 장점으로 볼 수도 있습니다.

<br>
 2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.
<hr> 

정적 팩터리 메서드를 사용하면 생성자처럼 API 설명에 명확히 드러나지 않으므로

정적 팩터리 메서드 클래스를 인스턴스화할 방법을 찾아야합니다.

그래서 보통 정적 팩터리 메서드를 만들 때 명명 규칙을 지킵니다.

### 2-5. 명명 규칙

* from

    매개 변수 하나를 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드

    ```
        Date date = Date.from(instant);
    ``` 

* of

    여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드

    ```
        LocalDate today = LocalDate.of(2022, 4, 9);
    ``` 

* valueOf

    from과 of의 더 자세한 버전

    ```
        String number = String.valueOf(123);
    ```

* instance, getInstance

    매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장

    ```
        Person person = Person.getInstance();
    ```

* create, newInstance

    instance, getInstance와 동일하지만, 매번 새로운 인스턴스를 생성해 반환함을 보장

    ```
        Person person = Person.newInstance();
    ```

* getType

    getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 사용

    ```
        FileStore fs = Files.getFileStore(path);
    ```

* newType

    newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 사용

    ```
        BufferedReader br = Files.newBufferedReader(path);
    ```

* type

    getType과 newType의 간결한 버전

    ```
        List<Brand> computer = Collections.list(brand);
    ```

## 3. 정리

정적 팩터리 메서드라는 용어의 의미와 함께 그것을 어떻게 사용하는지 예시를 통해 알아보았고

또한, 정적 팩터리 메서드를 사용함으로써 얻는 장점과 단점도 알아보았습니다.

정적 팩터리 메서드와 public 생성자는 각자의 쓰임새가 있고 장단점이 있기 때문에

적절한 위치에 두 객체 생성 방법을 적용하면 좋을 것 같습니다.
