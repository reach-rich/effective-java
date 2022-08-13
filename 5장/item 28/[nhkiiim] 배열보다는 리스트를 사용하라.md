# 배열보다는 리스트를 사용하라

### 1. 배열과 제네릭 타입의 차이
__1) 배열의 문제__
- 배열은 공변이다 (같이 변한다) (Super[], Sub[] 가 상하위 구조면 같이 변함)
- 하지만 제네릭은 불공변이다 (List<Type1\>, List<Type2\>)

```java
Object[] objectArray = new Long[1];
objectArray[0] = "타입이 달라 넣을 수 없다.";
```

```java
List<Object> ol = new ArrayList<Long>();
ol.add("타입이 달라 넣을 수 없다.");
```

- 둘 다 잘못된 코드지만 배열은 런타임에, 리스트는 컴파일 시에 알 수 있다!

<br>

__2) 배열은 실체화 된다__
- 배열은 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인한다
- 제네릭은 타입 정보가 런타입에는 소거된다
- 원소 타입을 컴파일 시에만 확인해 런타임에는 알 수 없다
- 소거는 제네릭으로 순조롭게 전환되도록 도와주는 것임

#
### 2. 배열은 제네릭타입, 매개변수화타입, 타입매개변수로 활용할 수 없다
- new List<E\>[], new List<String\>[], new E[] `(다 불가능)`
- 막은 이유는? 타입이 안전하지 않기 때문
- 이를 허용하면 컴파일러가 자동 생성한 형변환 코드에서 런타임에 ClassCastException이 발생하게 됨!

```java
List<String>[] stringLists = new List<String>[1];    // (1)
List<Integer> intList = List.of(42);                 // (2)
Object[] objects = stringLists;                      // (3)
Object[0] = intList;                                 // (4)
String s = stringLists[0].get(0);                    // (5)
```

- 제네릭 배열을 생성하는 (1)이 허용된다고 가정
- (3)에서 배열은 공변이니 List<String\>[]을 Object[]에 넣는 것에 문제가 없음
- (4)에서 제네릭은 소거 방식으로 구현되어 있기 때문에 List<Integer\>의 인스턴스를 Object 배열의 첫 원소로 저장
- 런타임에 List<Integer\> 인스턴스 타입은 단순한 List가 되고, List<Integer\>[] 타입은 List[]
- 따라서 (4)에서도 ArrayStoreException을 일으키지 X
- (5)에서 문제가 발생 (컴파일러는 꺼낸 원소를 자동으로 String으로 형변환, 하지만 Integer이므로 런타임에 ClassCastException이 발생)

<br>

- E, List<E\>, List<String\> 과 같은 타입을 실체화 불가 타입이라고 함
- 실체화 되지 않아 런타임에는 컴파일 타임보다 정보를 적게 가짐
- 소거 매커니즘 때문에 매개변수화 타입 가운데 실체화 될 수 있는 타입은 List<?>, Map<?,?>뿐!
- 배열을 비한정정 와일드카드 타입으로 만들순 있지만 굳이..?

#
### 3. 배열을 제네릭으로 만들 수 없어 귀찮은 상황
- 제네릭 타입과 가변인수 메서드를 함께 쓰면 경고 메세지를 받게 됨 (매개변수의 개수를 동적으로 지정해 줄 수 있게 되었는데 이 기능을 가변인자라고 함)
- 이 문제는 @SafeVarage 어노테이션으로 대처 가능

<br>

- 배열로 형변환할 때 제네릭 배열 생성 오류나 비검사 형변환 경고가 뜨는 경우 대부분 List<E\>를 활용하면 해결 됨
- 코드가 복잡해지고 성능은 저하되지만 타입 안전성과 상호 운용성은 좋아짐

<br>

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
- 생성자에서 컬렉션을 받는 예시
- 생성자에게 어떤 컬랙션을 넘기냐에 따라 다양하게 사용 가능
- 이 클래스를 사용하려면 반환된 object는 원하는 타입으로 변환 해야함
- 다른 타입의 원소가 들어있음 런타임에러

```java
public class Chooser<T> {
    private final T[] choiceArray;
    
    public Chooser(Collection<T> choices) {
        choiceArray = choices.toArray();
    }
    
    // choose 메서드는 그대로임
}
```
- 제네릭으로 바꾸면 경고가 막 뜬다 T가 무슨 타입인지 알 수 없으니 안전을 보장 할 수 없다는 것!
- 프로그램은 동작하지만 안전성은 없다
- 안전하다고 판단되면 배운 것처럼 어노테이션을 달아도 되긴한다 (하지만 애초에 경고를 없애라!)

<br>

- 비검사 형변환 경고를 제거하려면 배열 대신 리스트를 쓰면 된다!
- 코드양도 좀 늘고 좀 느릴 수 있지만 안전해진다! 가치 있다!!

```java
public class Chooser<T> {
    private final List<T> choiceList;
    
    public Chooser(Collection<T> choices) {
        choiceList = new ArrayList<>(choices);
    }
    
    public T choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceList.get(rnd.nextInt(choiceList.size()));
    }
}
```



