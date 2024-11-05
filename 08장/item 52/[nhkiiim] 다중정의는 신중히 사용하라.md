## 다중정의는 신중히 사용하라

### 다중정의(오버로딩)의 문제점
> 컬렉션, 집합, 리스트, 그 외로 구분하기 위한 프로그램

```java
public class CollectionClassifier {
    public static String classify(Set<?> s) {
        return "집합";
    }

    public static String classify(List<?> lst) {
        return "리스트";
    }

    public static String classify(Collection<?> c) {
        return "그 외";
    }

    public static void main(String[] args) {
        Collection<?>[] collections = {
                new HashSet<String>(),
                new ArrayList<BigInteger>(),
                new HashMap<String, String>().values()
        };

        for (Collection<?> c : collections)
            System.out.println(classify(c));
    }
}
```
- 컴파일 타임에 for문 안의 c는 항상 Collection<?> 타입

- 다중정의(오버로딩)된 `classify`중 어느 메서드를 호출할지는 컴파일 타임에 정해지기 때문에 이 프로그램은 "그 외" 만 출력

> 런타임에는 매번 타입이 달라지지만 호출할 메서드를 선택하는데 영향을 주진 못한다.

#
### 재정의과 다중정의
- __재정의(오버라이딩)한 메서드는 동적으로 선택__
- __다중정의(오버로당)한 메서드는 정적으로 선택__

> 메서드를 재정의 했다면 해당 객체의 런타입 타입이 어떤 메서드를 호출할지의 기준이 된다.
```java
class Wine {
    String name() { return "포도주"; }
}

class SparklingWine extends Wine {
    @Override String name() { return "발포성 포도주"; }
}

class Champagne extends SparklingWine {
    @Override String name() { return "샴페인"; }
}

public class Overriding {
    public static void main(String[] args) {
        List<Wine> wineList = List.of(
                new Wine(), new SparklingWine(), new Champagne());

        for (Wine wine : wineList)
            System.out.println(wine.name());
    }
}
```
> 하지만 다중정의된 메서드 사이에서는 객체의 런타임 타입은 전혀 중요하지 않다!!
 선택은 오직 __매개변수의 컴파일타임 타입__ 에 의해 결정된다.

#
### 다중정의 문제 해결 방법

#### 💡 instanceof 사용
- (정적 메서드를 사용해도 좋다면) instanceof로 명시적 검사 진행하기

```java
public static String classify(Collection<?> c) {
    return c instanceof Set  ? "집합" :
            c instanceof List ? "리스트" : "그 외";
}
```
#### 🎯 안전하고 보수적으로 가기 위해서는 매개변수 수가 같은 다중정의는 만들지 말자
- 가변인수(`String...str`)를 사용하는메서드라면 다중정의 사용 금지

#### 🎯 메서드 이름 분리
- 어려운 일도 아니니 다중정의보다는 메서드의 이름을 다르게 지어주자
- ObjectOutputStream의 `writeBoolean(boolean)`, `writeInt(int)`, `writeLong(long)`과 같이 메서드 이름 분리 (`read()` 메서드와 짝 맞추기도 좋음)

#### 🎯 정적 팩터리 사용
- 생성자는 이름을 다르게 지을 수 없으니 정적 팩터리라는 대안을 사용 가능


#
### 다중정의가 혼돈을 일으키는 상황
> 재정의한 메서드는 프로그래머가 기대한 대로 동작하나 다중정의한 메서드는 기대를 벗어날 수 있으므로 특히 API 설계 시 주의가 필요하다.

__(1) 자바5 에서 오토박싱 도입 이후 혼란 발생__
- (`List<E>`의 `remove(Object)`, `remove(int)` - Integer형 숫자 제거를 int형 index 제거로 혼돈 가능)

__(2) 자바 8에서 람다와 메서드 참조 추가 이후 혼란 발생__
```java
new Thread(System.out::println).start();
```
```java
ExcutorService exec = Executors.newCachedThreadPool();
exec.submit(System.out::println);
```
- 두번째 소스만 컴파일 오류 발생 (submit 다중정의 메서드 때문)

> 메서드 다중정의 시 서로 다른 함수형 인터페이스라도 같은 위치의 인수로 받으면 안된다.


#

> 다중정의를 허용한다고 꼭 다중정의를 활용하라는 건 아니다. 명확하게 설계하자


