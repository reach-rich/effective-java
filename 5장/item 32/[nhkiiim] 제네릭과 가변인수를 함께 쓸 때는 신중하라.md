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
### 2. 
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
- 이 메세드에서는 형변환하는 곳이 보이지 않아도 ClassCastException
