# 아이템 2: 생성자 매개변수가 많은 경우에 빌더 사용을 고려해 볼 것

<br>

## 1. 생성자 사용

```java
NutritionFacts cocaCola =
new NutritionFacts(240, 8, 100, 0, 35, 27);
```

- 생성자가 길어지고 각각의 매개변수가 무엇을 의미하는지 알 수 없음
- 필요 없는 매개변수를 넘겨야하기도 한다

<br>

## 2. 자바빈 사용

```java
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```

- 세터를 사용하는 방법
- 어떤 매개변수를 세팅하는지 명확하게 이해 가능
- 코드가 길어짐
- 자바빈이 안정적이지 않은 상태에서 사용하게 됨


<br>

## 해결책 3: 빌더

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
.calories(100).sodium(35).carbohydrate(27).build();
```

```java
public abstract class Pizza {

    public enum Topping {
        HAM, MUSHROOM, ONION, PEEPER, SAUSAGE
    }

    final Set<Topping> toppings;

    abstract static class Builder<T extends  Builder<T>> { 
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);

        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }

        abstract Pizza build(); 

        protected abstract T self();
    }

    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone();
    }

}
```

```java
public class NyPizza extends Pizza {

    public enum Size {
        SMALL, MEDIUM, LARGE
    }

    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }


        @Override
        public NyPizza build() {
            return new NyPizza(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }

    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }
}
```

```java
public class Calzone extends Pizza {

    private final boolean sauceInside;

    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sauseInside = false;

        public Builder sauceInde() {
            sauseInside = true;
            return this;
        }

        @Override
        public Calzone build() {
            return new Calzone(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }

    private Calzone(Builder builder) {
        super(builder);
        sauceInside = builder.sauseInside;
    }

}
```



- 생성자와 자바빈이 가지는 장점을 다 가질 수 있음
- 필수적인값과 부가적인 값들을 지정해 줄 수 있음
- 빌드의 생성자에서 유효성 확인도 가능
- 객체를 만들기 전에 빌더를 만들어야해서 성능 측면에서 문제가 될 수도 있음(백기선 선생님 : 성능 영향 사실 미미함)
- 생성자를 사용하는것보다 코드가 길어짐
- 매개변수가 많거나 늘어날 가능성이 있는 경우 사용하는 것이 좋음 



<br>
