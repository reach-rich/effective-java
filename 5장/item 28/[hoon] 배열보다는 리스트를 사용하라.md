## 1. 들어가기

우리가 흔히 사용하는 배열과 제네릭은 크게 두 가지의 차이점이 있습니다.

한 번 자세하게 알아보도록 하겠습니다.

## 2. 배열 VS 제네릭

1. **공변**

   공변은 타입 A가 B의 하위 타입일 때 Array[A]는 Array[B]의 하위 타입 임을 나타내는 개념입니다.

   배열은 공변이기 때문에 `Array[Parent] parent = new Array[Child]`가 성립됩니다.

   하지만 제네릭은 불공변이라서 `List<Parent> parent = new List<Child>`가 성립되지 않습니다.

2. **실체화 여부**

   실체화는 런타임 시에도 원소의 타입을 인지하는지의 개념입니다.

   배열은 실체화되기 때문에 Long 배열에 String을 넣으려고 하면 ArrayStoreException이 발생합니다.

   하지만 제네릭은 실체화되지 않고 소거되기 때문에 런타임 시에 타입을 알 수 없습니다.

이상의 두 차이로 배열과 제네릭은 잘 어울러지지 못합니다.

그 예로 실체화 불가 타입(제네릭 타입, 매개변수화 타입, 타입 변수)은 배열로 사용할 수 없습니다.

`new List<E>[]`, `new List<String>[]`, `new E[]`

## 3. 제네릭 배열을 사용하지 못하는 이유

제네릭 배열을 만들지 못하게 막은 이유가 무엇일까요?

예시로 알아보겠습니다.

```java
  List<String>[] stringLists = new List<String>[1];   // (1)
  List<Integer> intList = List.of(42);                // (2)
  Object[] objects = stringLists;                     // (3)
  objects[0] = intList;                               // (4)
  String s = stringLists[0].get(0);                   // (5)
```

(1)은 원래 컴파일 오류이지만, 허용된다고 가정해 보겠습니다.

(2)는 42라는 원소를 가진 List입니다.

(3)은 (1)에서 생성한 List\<String> 배열을 Object 배열에 할당합니다. (공변 변환)

(4)는 (2)에서 생성한 List\<Integer>를 Object 배열의 첫 원소로 저장합니다.

자, (5)번을 위해 지금까지 빌드업 해왔습니다.

원래 List\<String> 인스턴스만 담겠다고 선언한 stringLists 배열의 첫 번째에는 무엇이 들어있을까요?

바로 (4)에서 넣은 List\<Integer>가 들어있습니다.

그래서 (5)는 String 타입에 Integer를 넣으려고 하기 때문에 ClassCastException이 발생합니다.

이런 일을 사전에 방지하려면 컴파일 오류를 내야 합니다.

## 4. 비검사 경고 제거 방법

배열로 형변환할 때 제네릭 배열 생성 오류나 비검사 형변환 경고가 뜨는 경우가 있습니다.

이 경우에는 배열 E[] 대신 리스트 List\<E>를 사용하면 됩니다.

예시를 통해 알아봅시다.

```java
  public class Chooser {
    private final Object[] choiceArray;

    public Chooser(Collection choices) {
      choiceArray = choices.toArray();
    }

    public Object choose() {
      Random rnd = ThreadLocalRandom.current();
      return choiceArray[rnd.nextInt(choiceArray.length)];
    }
  }
```

해당 예시는 컬렉션 안의 원소 중 하나를 무작위로 선택해 반환합니다.

이 클래스의 문제점이 무엇일까요?

현재 choiceArray 배열은 Object 타입이기 때문에 사용하려면 원하는 타입으로 형변환을 해야합니다.

그럼 이 예시를 제네릭으로 변경해보겠습니다.

```java
  public class Chooser<T> {
    private final T[] choiceArray;

    public Chooser(Collection<T> choices) {
      choiceArray = (T[]) choices.toArray();
    }

    public Object choose() {
      Random rnd = ThreadLocalRandom.current();
      return choiceArray[rnd.nextInt(choiceArray.length)];
    }
  }
```

하지만 다음의 경고를 볼 수 있습니다.

```
  Chooser.java:9: warning: [unchecked] unchecked cast
        choiceArray = (T[]) choices.toArray();
                                           ^
  required: T[]
  found:    Object[]
  where T is a type-variable:
    T extends Object declared in class Chooser
```

T[]가 무슨 타입인지 알 수 없어 형변환이 런타임에도 안전한지 보장할 수 없다는 의미입니다.

Item 27에서 비검사 경고는 제거해야 한다고 했습니다.

그럼, 어떻게 해결할 수 있을까요?

그저 배열 대신 리스트를 사용하면 됩니다.

```java
  public class Chooser<T> {
    private final List<T> choiceList;

    public Chooser(Collection<T> choices) {
      choiceList = new ArrayList<>(choices);
    }

    public Object choose() {
      Random rnd = ThreadLocalRandom.current();
      return choiceArray.get(rnd.nextInt(choiceArray.size()));
    }
  }
```

리스트를 사용한다면 비록, 코드양이 늘고 성능이 다소 떨어지지만 타입 안전성을 보장할 수 있습니다.

## 5. 정리

이번 포스트는 배열과 제네릭에 대해서 알아보았습니다.

배열과 제네릭은 크게 **공변**과 **실체화 여부**에서 차이점이 있었습니다.

그렇기 때문에 둘을 섞어 사용하는 것은 쉽지 않습니다.

만약, 둘을 섞어 사용하다가 컴파일 오류나 경고를 만나게 된다면, 배열 대신 리스트로 사용해봅시다.