## 전통적인 for문 보다는 for-each 문을 사용하라

> 전통적인 for문으로 컬렉션을 순회하는 코드

```java
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
  Element e = i.next();
}
```

> 전통적인 for문으로 배열을 순회하는 코드

```java
for (int i=0; i < a.length; i++) {
  ... 
}
```

- for문이 while문 보단 낫지만(지역변수 범위 줄임) 가장 좋은 방법은 아님
- 인덱스 변수는 코드를 지저분하게 할 뿐 진짜 필요한 변수가 아님
- 인덱스 변수처럼 사용하는 요소들이 많아지면 오류 발생 가능성이 높아짐

#
### for-each문

- 반복자와 인덱스 변수를 사용하지 않아 코드가 깔끔하고 오류 확률이 낮아짐
- 하나의 관용구로 컬렉션과 배열을 모두 사용 가능해 어떤 컨테이너를 다루는지 신경쓰지 않아도 됨

```java
for (Element e : elements) {
  ... // elements 안의 각 원소 e 에 대해 처리
}
```

- 반복 대상이 컬렉션이든 배열이든 뽀이치문을 사용해도 속도는 그대도 유지

#
### for-each문을 사용할 수 없는 세가지 상황

#### 1) 파괴적인 필터링
- 컬렉션을 순회하면서 선택한 원소를 제거해야한다면 반복자의 remove 메서드 호출 필요
- 자바 8부터는 removeIf 메서드를 사용해 컬렉션을 명시적으로 순회하는 일을 피할 수 있음

#### 2) 변형
- 리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 변경해야할 경우 인덱스 사용 필요

#### 3) 병렬 반복 
- 여러 컬렉션을 병렬로 순회해야한다면 각각의 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어 필요

#
### Iterable 인터페이스 구현해두기
- Iterable 인터페이스를 구현해두면 뽀이치문을 사용 가능
- 원소의 묶음을 표현하는 타입을 작성 할 때 Iterable을 구현하는 걸 고민해보자!
