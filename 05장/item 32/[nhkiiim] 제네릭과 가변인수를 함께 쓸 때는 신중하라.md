# 제네릭과 가변인수를 함께 쓸 때는 신중하라

### 1. 가변인수와 제네릭은 안맞아

- 가변인수와 제네릭은 자바 5 때 함께 추가되어 서로 잘 어우러지리라 생각한다면 천만의 말씀 만만의 콩떡
- 가변인수는 메서드에 넘기는 인수의 개수를 클라이언트 조절할 수 있게 해주나, 허점이 있지!
- 가변인수 메서드르 호출하면 가변인수르 담기 위한 배열이 자동으로 하나 만들어짐 
- 하지만 내부로 감춰야하는 이 배열을 클라이언트에게 노출하는 문제 발생!
- varargs 매개변수에 제네릭이나 매개변수화 타입이 포함되면 알기 어려운 컴파일 경고 발생!
  - varargs : 자바 1.5에서 가변 인자 메서드(Variable arity method)라고 부르는 varargs 메서드가 추가, 지정된 자료형의 인자를 0개 이상 받을 수 있음
  - (String ...srts)처럼 사용

- 제네릭과 매개변수화 타입은 실체화 되지 않기 때문에 런타임시에 컴파일 타임보다 정보를 적게 담고 있음
- 그래서 메세드를 선언할 때 실체와 불가 타입(제네릭)으로 varags를 선언하면 컴파일 경고!

#
### 2. 제네릭 varargs 배열 매개변수에 값을 저장하지 말자
- 매개변수화 타입의 변수가 타입이 다른 객체를 참조하면 힙 오염 발생!??
- 컴파일러가 자동 생성한 형변환도 실패 -> 제네릭 타입이 시스템이 약속한 타입 안정성이 흔들림

<br>

- 힙 오염 예시 (정상적으로 실행은 됨)
```java
ArrayList<String> arrayList = new ArrayList<>();
arrayList.add("String1");

Object obj = arrayList;

ArrayList<Integer> arrayList2 = (ArrayList<Integer>)obj;
arrayList2.add(new Integer(100));
```

<br>

- 암튼 제네릭과 varargs를 혼용하면 타입 안전성이 깨짐

```java 
static void dangerous(List<String>... stringLists) {
    List<Integer> intList = List.of(42);
    Object[] objects = stringLists;
    objects[0] = intList; // 힙 오염 발생
    String s = stringLists[0].get(0); // ClassCastException
}
```
- 이 메세드에서는 형변환하는 곳이 보이지 않아도 ClassCastException 발생
- 마지막줄에 컴파일러가 생성한 보이지 않는 형변환이 생기기 때문!
- 그러므로 제네릭 varargs 배열 매개변수에 값을 저장하는 것은 안전하지 않음
- 제네릭 배열을 프로그래머사 직접 생성하는 건 안되지만 제네릭 varargs 매개변수를 받는 메서드를 선언할 수 있게 한 이유
  - 제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 메서드가 실무에서 매우 유용

#
### 3. 제네릭 가변인수 메서드의 경고
- 자바 7전에는 제네릭 가변인수 메서드의 경고 숨기는 방법 없었다 
- @SuppressWarning("unchecked")를 달아 경고 숨겼어야 했음!

<br>

- 자바 7에서는 @SafeVarargs 애너테이션이 추가
- 제네릭 가변인수 메서드 작성자가 클라이언트 측에서 발생하는 경고를 숨길 수 있음
- @SafeVarargs 애너테이션은 메서드 작성자가 그 메서드가 타입 안전함을 보장하는 장치
- 컴파일러는 이걸 믿고 경고를 하지 않음
- 하지만 메서드가 안전한게 확실하지 않다면 절대 @SafeVarags 달면 안됨

#
### 4. 제네릭 가변인수 메서드가 안전한지 확신할 수 있는 방법
- 가변인수 메서드를 호출할 때 varargs 매개변수를 담는 제네릭 배열 생성됨
- 메서드가 이 배열에 아무것도 저장하지 않고 그 배열의 참조가 밖으로 노출되지 않는다면 타입 안전!
- 즉!! 순수하게 인수들을 전달하는 일만 한다면(목적대로) 그 메서드는 안전

<br>

- 자신의 제네릭 매개변수 배열의 참조 노출 : 안전하지 않다!

```java
static <T> T[] toArray(T... args) {
    return args;
}
```

- 메서드가 varags 매개변수 배열을 그대로 반환해 힙오염 전이할 수 있음

```java
static <T> T[] pickTwo(T a, T b, R c) {
  switch(ThreadLocalRandom.current.nextInt(3)){
    case 0 : return toArray(a, b);
    case 1 : return toArray(a, c);
    case 2 : return toArray(c, b);
  }
  throw new AsserionErrors(); //도달 불가 (문제 없음)
}
```

- 이 메서드를 본 컴파일러는 T 인스턴스 2개를 담을 varargs 생성
- 배열은 Object[]가 되고 반환

```java
public static void main(String[] args){
  String[] attr = pickTwo("좋은", "빠른", "저렴한"); 
}
```
- 컴파일은 제대로 되지만 컴파일러가 Object[] 를 String[]으로 형변환 하려는 코드가 자동 생성되고
- 형변화 될 수 없기 때문에 ClassCastException 발생

- 제네릭 varargs 매개변수 배열에 다른 메서드가 접근하면 안전하지 않음

<br>

- 제네릭 varargs 매개변수 배열에 다른 메서드가 접근해도 되는 예외 단 두가지
- @SafeVarargs로 제대로 어노테이트된 또 다른 varargs에 넘기는 것은 가능
- 그저 이 배열 내용의 일부를 호출만 하는 (varargs를 받지 않는) 일반 메서드에 넘기기 가능

#
### 5. 제네릭 varargs 매개변수를 안전하게 사용하는 메서드
```java
@SafeVarargs
static <T> List<T> flatten(List<? extends T>... lists) {
    List<T> result = new ArrayList<>();
    for (List<? extends T> list : lists)
        result.addAll(list);
    return result;
}
```
- 임의 개수의 리스트를 받아 받은 순서대로 하나의 리스트로 옮겨 반환 
- @SafeVarargs가 있어 선언하고 사용하는데 경고 X


#
### 6. @SafeVarargs를 사용해야할 때
- 제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 모든 메서드에 @SafeVarargs를 달아라 (안전하지 않은건 당연히 안됨)
- 통제할 수 있는 메서드 중 제네릭 varargs 매개변수를 사용하며 힙 오염 경고가 뜨는 메서드가 있면 안전한지 점검해라
- 다음 두 조건을 모두 만족하는 제네릭 varargs 메서드는 안전하다! 둘 중 하나라도 어기면 수정 ㄱㄱ

<br>

- __varargs 매개변수 배열에 아무것도 저장하지 않는다.__
- __그 배열(혹은 복제본)을 신뢰할 수 없는 코드에 노출하지 않는다.__


#
### 7. @SafeVarargs 어노테이션이 유일한 정답은 아니다!
- 실체 배열인 varargs를 List 매개변수로 바꿀 수도 있다

```java
@SafeVarargs
static <T> List<T> flatten(List<List<? extends T)> lists) {
    List<T> result = new ArrayList<>();
    for (List<? extends T> list : lists)
        result.addAll(list);
    return result;
}
```

- 정적 팩터리 메서드인 List.of를 활용하면 임의 개수의 인수를 넘길 수 있당
- List.of(1,2,3) -> @SafeVarargs 달려있음
- 우리가 직접 어노테이션 안달아고 되고 컴파일러가 안전성 검증도 할 수 있어 좋은 방법
- 코드가 지저분해지고 좀 느려질 순 있음^^

<br>

- toArray처럼 varargs 메서드를 안전하게 사용하지 못하는 상황에서도 활용 가능


```java
static <T> T[] pickTwo(T a, T b, R c) {
  switch(ThreadLocalRandom.current.nextInt(3)){
    case 0 : return List.of(a, b);
    case 1 : return List.of(a, c);
    case 2 : return List.of(c, b);
  }
  throw new AsserionErrors();
}
```

```java
public static void main(String[] args){
  List<String> attr = pickTwo("좋은", "빠른", "저렴한"); 
}
```
