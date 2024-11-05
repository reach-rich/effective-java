# equals는 일반 규약을 지켜 재정의하라

### 1. equals 재정의의 교훈 : 가마니 있으면 중간은 간다
- 재정의하기 쉬워 보이지만 함정들이 있어서 자칫하면 끔찍깜찍해짐
- 문제를 회피하는 가장 쉬운 방법은 아예 재정의 하지 않는 것

- 다음 상황 중 하나에 해당하면 그냥 두는게 낫다!

<br>

1. 각 인스턴스가 본질적으로 고유할 때
- 값 클래스(Integer나 String처럼 값을 표현하는 클래스)가 아닌 동작하는 개체를 표현하는 클래스
- ex) Thread 가 좋은 예시!!

2. 인스턴스의 '논리적 동치성'을 검사할 일이 없을 때
- ex) java.util.regax.Pattern은 equals를 재정의해 두 Pattern의 인스턴스가 같은 정규표현식인지 확인

3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞을 때
- ex) Set 구현체는 AbstractSet이 구현한 equals를 상속, List는 AbstractList 상속, Map은 AbstractMap 상속

4. 클래스가 private이나 package-private이고 equals를 호출할 일이 없을 때
- equals가 실수로라도 호출되는 걸 막을 수도 있음

```java
@Override public boolean equals(Object o) {
	throw new AssertionError(); // 호출 금지!
}
```

#
### 2. equals를 재정의 해야할 때

- 객체 식별성(object identity; 두 객체가 물리적으로 같은가)이 아닌 '논리적 동치성'을 확인해야 함
  - 논리적 동치성 참고 -> https://javanitto.tistory.com/9

- 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의 되지 않았을 때 
- 주로 값 클래스가 해당됨 (Integer, String)

- 두 값 객체를 equals로 비교하는 경우, 객체가 같은지가 아니라 값이 같은지를 알고싶을 것!
- equals가 논리적 동치성을 확인하도록 재정의하면, 값 비교는 물론 Map의 키와 Set의 원소로 사용 가능
- 하지만 값 클래스여도 같은 인스턴스가 둘 이상 만들어지지 않는 인스턴스 통제 클래스라면 재정의하지 않아도 됨~! `당연한 말`
  - Enum이 여기에 해당 (2개 이상의 객체가 만들어지지 않으니까~)


#
### 3. equals 메서드 재정의 일반 규약을 따라라~
- 반사성(reflexivity) : null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true다.
- 대칭성(symmetry) : null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)가 true면 y.equals(x)도 true다.
- 추이성(transitivity) : null이 아닌 모든 참조 값 x, y, z에 대해, x.equals(y)가 true이고, y.equals(z)도 true면 x.equals(z)도 true다.
- 일관성(consistency) : null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)를 반복해서 호출하면 항상 true이거나 false다.
- null-아님 : null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다.

<br>
- 참고 : https://velog.io/@lychee/이펙티브-자바-아이템-10.-equals는-일반-규약을-지켜-재정의-하라
- 이 규칙 좀 어려워도 절대 그냥 넘어가지마라!

- 이 규약을 어기면 프로그램이 이상해짐


#
### 4. 제대로 equals 구현하기
1. 연산자를 사용해 입력이 자기 자신의 참조인지 확인하자
	- 자기 자신이면 true를 반환
	- 단순한 성능 최적화용으로 비교 작업이 복잡한 상황일 때 값어치를 함!

2. instanceof 연산자로 입력이 올바른 타입인지 확인하자
	- 가끔 해당 클래스가 구현한 특정 인터페이스를 비교할 수도 있음
	- 이런 인터페이스를 구현한 클래스라면 equals에서 (클래스가 아닌) 해당 인터페이스를 사용해야한다.
	- ex) Set, List, Map, Map.Entry 등 컬렉션 인터페이스들

3. 입력을 올바른 타입으로 형변환 하자
	- 2번에서 instanceof 연산자로 입력이 올바른 타입인지 검사 했기 때문에 이 단계는 100% 성공!!

4. 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사하자
	- 모두가 일치하면 true를 반환, 하나라도 다르면 안됨!!
	- 인터페이스 사용했다면 그 인터페이스 메서드 사용해야함!!


#
### 5. 또또 주의할 것
- 기본 타입은 == 연산자 비교
- 참조 타입은 equals 메서드로 비교
- float, double 필드는 정적 메서드 Float.compare(float, float)와 Double.compare(double, double)로 비교
  - Float.equals(float)나 Double.equals(double)은 오토 박싱을 수반해 성능상 좋지 않다.

- 배열 필드는 원소 각각을 지침대로 비교
  - 모두가 핵심 필드라면 Arrays.equals()를 사용
- null 정상값 취급 방지 Object.equals(object, object)로 비교하여 NullPointException 발생을 예방!

- 비교하기 복잡한 필드를 가진 클래스는 필드의 표준형(canonical form)을 저장한 후 표준형끼리 비교
- 필드의 비교 순서는 equals 성능을 좌우함

