## 1. 들어가기

생성자는 객체를 생성하는 가장 기본적인 방법입니다.

그리고 생성자 오버로딩을 통해 시그니처가 다른 여러 생성자를 만들 수 있습니다.

그럼 매개변수가 여러 개 있다면 어떻게 구현할 수 있을까요?

이럴 때 개발자들은 보통 '점층적 생성자 패턴' 을 사용했습니다.

점층적 생성자 패턴이 뭘까요?

예시를 보겠습니다.

```
    public class Person {
        private String name;
        private int age;
        private String job;
        private String hobby;

        Person(String name, int age) {
            this(name, age, null);
        }

        Person(String name, int age, String job) {
            this(name, age, job, null);
        }

        Person(String name, int age, String job, String hobby) {
            this.name = name;
            this.age = age;
            this.job = job;
            this.hobby = hobby;
        }
    }
```

이렇게 매개변수를 하나씩 점층적으로 늘려가 생성자를 구현하는 패턴이 '점층적 생성자 패턴' 입니다.

하지만, 매개변수가 10개, 20개가 된다면 어떻게 될까요?

분명 생성자만 작성하다가 지칠 것입니다.

즉, 점층적 생성자 패턴도 쓸 수는 있지만, 코드를 작성하기 어렵고 가독성도 떨어집니다.

심지어 위의 예시 코드에서 job과 hobby의 값을 반대로 넣어도

컴파일러는 알지 못해 런타임에 엉뚱한 동작을 하게 됩니다.

그럼 방법이 없는걸까요?

이번에는 또다른 대안인 Java Beans 패턴에 대해 알아보겠습니다.

Java Beans 패턴은 먼저, 매개변수가 없는 생성자로 객체를 만든 후에

setter를 호출해 원하는 매개변수의 값을 설정하는 방식입니다.

이것도 예시를 한번 보겠습니다.

```
    public class Person {
        private String name;
        private int age;
        private String job;
        private String hobby;

        public Person() {}

        public void setName(String name)    { this.name = name; }
        public void setAge(int age)         { this.age = age; }
        public void setJob(String job)      { this.job = job; }
        public void setHobby(String hobby)  { this.hobby = hobby; }
    }

    public static void main(String[] args) {
        Person person = new Person();

        person.setName("이상훈");
        person.setAge(28);
        person.setJob("developer");
        person.setHobby("playing piano");
    }
```

확실히 점층적 생성자 패턴에 비해 인스턴스를 만들기 쉽고, 가독성이 좋아졌습니다.

하지만, 객체 하나를 만들기 위해서는 여러 메서드를 호출해야하고

객체가 완전히 생성되기 전까지는 일관성이 무너진 상태에 놓이게 됩니다.

이 단점을 보완하기 위해 객체를 완전히 생성하기 전까지는 사용하지 못하도록

freezing을 사용하는 방법이 있지만

이는 다루기 어려워 실전에서는 거의 쓰이지 않습니다.

그럼 객체를 생성할 때 메서드를 적게 호출하면서 일관성이 유지되는 방법이 없는걸까요?

지금부터 또 하나의 대안 Builder에 대해서 알아봅시다!

## 2. Builder

### 2-1. 의미<br>

Builder가 무엇일까요?

먼저, Builder의 사전적 의미를 알아보면 "건축가" 라는 뜻입니다.

다음 예시를 보면 왜 Builder라는 용어를 사용하는지 알 수 있을 것입니다.

```
    public class Person {
        private String name;
        private int age;
        private String job;
        private String hobby;

        public static class Builder {
            private String name = null;
            private int age = 0;
            private String job = null;
            private String hobby = null;

            public Builder(String name, int age) {
                this.name = name;
                this.age = age;
            }

            public Builder job(String job) {
                this.job = job;
                return this;
            }

            public Builder hobby(String hobby) {
                this.hobby = hobby;
                return this;
            }

            public Person build() {
                return new Person(this);
            }
        }

        private Person(Builder builder) {
            this.name = builder.name;
            this.age = builder.age;
            this.job = builder.job;
            this.hobby = builder.hobby;
        }
    }

    public static void main(String[] args) {
        Person person = new Person.Builder("이상훈", 28)
                                  .job("develop")
                                  .hobby("playing piano")
                                  .build();
    }
```

주의깊게 봐야할 곳은 main 메서드 부분입니다.

부품(job, hobby)을 하나씩 조립하는 느낌이 들지 않나요?

Builder의 setter 메서드들은 Builder 자기 자신을 반환하기 때문에

예시와 같이 연쇄적으로 호출할 수 있습니다.

이런 방식을 메서드 호출이 흐르듯 연결된다는 뜻으로

Fluent API 또는 Method Chaining 이라고 합니다.

### 2-2. 예시<br>

Builder는 계층적으로 설계된 클래스와 함께 쓰기에 좋습니다.

예시를 통해 알아보겠습니다.

```
/* Sushi.class */

    public abstract class Sushi {
        public enum Classification { NORMAL, ABURI, MAKI }
        Classification classification;

        abstract static class Builder<T extends Builder<T>> {
            Classification classification;
            public T addClassification(Classification classification) {
                this.classification = classification;
                return self();
            }

            abstract Sushi build();

            protected abstract T self();
        }

        Sushi(Builder builder) {
            this.classification = builder.classification;
        }
    }
```

```
/* NormalSushi.class */

    public class NormalSushi extends Sushi {
        public enum Neta { SAKE, ENGAWA, MAGURO }
        private Neta neta;

        public static class Builder extends Sushi.Builder<Builder> {
            private Neta neta;

            public Builder (Neta neta) {
                this.neta = Objects.requireNonNull(neta);
            }

            @Override
            public NormalSushi build() {
                return new NormalSushi(this);
            }

            @Override
            protected Builder self() {
                return this;
            }
        }

        private NormalSushi(Builder builder) {
            super(builder);
            neta = builder.neta;
        }
    }
```

```
/* AburiSushi.class */

    public class AburiSushi extends Sushi {
        public enum Neta { SAKE, EBI, HOTATEGAI, WAGYU }
        public enum Source { MAYO, HONEYMAYO, SOY, GARLIC }

        private Neta neta;
        private Source source;

        public static class Builder extends Sushi.Builder<Builder> {
            private Neta neta;
            private Source source;

            public Builder (Neta neta, Source source) {
                this.neta = Objects.requireNonNull(neta);
                this.source = Objects.requireNonNull(source);
            }

            @Override
            public AburiSushi build() {
                return new AburiSushi(this);
            }

            @Override
            protected Builder self() {
                return this;
            }
        }

        private AburiSushi(Builder builder) {
            super(builder);
            neta = builder.neta;
            source = builder.source;
        }
    }
```

해당 예시는 예전에 스시집에서 아르바이트했던 경험을 토대로 만들어봤습니다.

Sushi 클래스는 스시의 종류(기본 스시, 아부리 스시, 군함)를 결정하는 추상 클래스입니다.

NormalSushi 클래스는 네타(생선회)를 매개변수로 받아 기본 스시로 만드는 구체 클래스이고,

AburiSushi 클래스는 네타 + 소스를 매개변수로 받아 아부리 스시로 만드는 구체 클래스입니다.

각 하위 클래스의 Builder가 정의한 build 메서드는 해당하는 구체 하위 클래스를 반환합니다.

즉, NormalSushi 클래스의 build 메서드는 기본 스시를 반환하고

AburiSushi 클래스의 build 메서드는 아부리 스시를 반환한다는 것이죠.

이제, 스시 장인이 되어 스시를 만들어볼까요?

저는 둘 다 좋아하니깐 둘 다 만들어볼게요!

먼저, 연어 스시를 만들어보겠습니다.

```
    NormalSushi SakeNormalSushi = new NormalSushi.Builder(NormalSushi.Neta.SAKE)
                                                 .addClassification(Sushi.Classification.NORMAL)
                                                 .build();
```

Normal 스시를 선택해 Builder의 매개변수인 네타를 연어로 선택해 연어 스시를 만들었습니다.

다음은 와규 스시를 만들어보겠습니다.

```
    AburiSushi WagyuAburiSushi = new AburiSushi.Builder(AburiSushi.Neta.WAGYU, AburiSushi.Source.SOY)
                                               .addClassification(Sushi.Classification.ABURI)
                                               .build();
```

이번에는 Aburi 스시를 선택해

Builder의 매개변수인 네타와 소스를 와규와 간장으로 선택해 와규 아부리 스시를 만들었습니다.

다음과 같이 Builder를 사용하면

코드의 가독성이 높아질 뿐만 아니라, 객체가 생성되기 전까지 일관성을 유지할 수 있습니다.

Builder는 상당히 유연합니다.

여러 객체를 순회하면서 만들 수 있고, 매개변수에 따라 다른 객체를 만들 수 있습니다.

하지만, 이런 Builder도 단점이 있습니다.

첫 번째로, 객체를 만들기 전 Builder를 만들어야 합니다.

Builder의 생성 비용이 크지는 않지만,

성능에 민감한 상황에서는 문제가 될 수 있습니다.

두 번째로, 점층적 생성자 패턴보다 매개변수가 작은 경우, 효율적이지 않을 수 있습니다.

하지만 API는 시간이 지날수록 매개변수가 많아지는 경향이 있기에

유지보수를 고려했을 때, Builder를 사용하는 것이 더 나을 수도 있습니다.

## 3. 정리

이번 포스트에서는 생성자를 통해 객체를 생성하는 방법에 대해서 알아보았습니다.

그 중 특히 점층적 생성자 패턴보다 가독성이 좋으면서

Java Beans보다 훨씬 안전한 Builder에 대해서 알아보았습니다.

그렇지만 Builder도 성능의 문제가 있을 수 있기 때문에 상황에 맞게 사용하면 좋겠습니다.
