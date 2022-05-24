# equals를 재정의하려거든 hachCode도 재정의하라
### 1. equals를 재정의한 클래스 모두에서 hashCode도 재정의해야 함!
- 재정의 하지 않으면 hashCode 일반 규약을 어기게 됨!

- 그러면 HashMap이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제가 생김

<br>

`Object 명세서에서 발췌한 규약`
> - equals 비교에 사용되는 정보가 변경되지 않았다면 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇 번을 호출해도 항상 같은 값을 반환해야 함 
>   (단, 애플리케이션을 다시 실행한다면 달라져도 무관)
> - equals(Object)가 두 객체를 같다고 판단했따면, 두 객체의 hashCode는 똑같은 값을 반환해야함
> - equals(Object)가 두 객체를 다르게 판단했더라도 구 객체의 hashCode가 다를 필요는 없음 
>   (단, 다른 값을 반환해야 해시테이블의 성능이 좋아짐)

<br>

- 크게 문제가 되는 조항은 2번!
  - 논리적으로 같은 객페는 같은 해시코드를 반환해야함
  - 논리적으로 같아도 Object의 기본 hashCode는 이를 전혀 다르다고 판단해 규약과 달리 서로 다른 값을 반환해버림

<br>

`요약 : 그니까 물리적으로 같은 객체는 같은 주소를 가져야한다 이 말씀 -> 그러니 equals 재정의 할 때 주소도 맞춰조라~!!`


#
### 2. 예시와 함께 알아보자

```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707,867,5309),"제니");

m.get(new PhoneNumber(707,867,5309)); // null 반환
```

- 2개의 PhoneNumber 인스 턴스가 사용됨
- put할 때, get할 때 두번 다 객체를 새로 만들었기 떄문에 룰 넘버 2 위배됨
- 그래서 엉뚱한 결과인 null이 나오는 것  (논리적으로 동치여도 물리적으로 다름)

- 심지어 두 객체를 같은 버킷에 담아도 같은 결과를 가짐
- 해시맵은 해시코드가 다른 엔트리끼리는 동치성 비교를 시도조차 하지 않도록 최적화 되어 있기 때문!




#
### 3. 절대 따라하지 마시오 : (적법하지만) 최악의 hashCode 구현 `절대 사용 금지!`

```java
@Override
public int hashCode() {return 42;}
```

- 동치인 모든 객체에서 같은 해시코드를 반환하게 해주는 소스코드

- 하지만 아주 끔찍하지 : 모든 객체에게 같은 값을 내어줘서 모든 객체가 해시테이블의 버킷 하나에 담겨 -> `Linked List`형태가 됨
- 연결 리스트처럼 동작하면 명균 수행 시간이 O(1)인 해시 테이블이 O(n)으로 느려짐
- 객체가 많아지면 도저히 쓸 수 가 없는 쓰레기가 되어버림 ㅠㅜ


#
### 4. 좋은 hashCode를 작성하는 방법

- 좋은 해시 함수라면 서로 다른 인스턴스에 다른 해시코드를 반환해야함 (룰 넘버 3)

- 이상적인 해시 함수는 주어진 (서로 다른) 인스턴스들을 32비트 정수 범위에 균일하게 분배해야함
- 이상을 완벽히 실현하기는 어렵지만 비슷하게 만들기는 어렵지 않음


<br>

`좋은 해시코드를 작성하는 간단한 요령~!`
- int 변수인 result를 선언한 후 값을 c로 초기화한다.
    - 이 때, c는 해당 객체의 첫번째 핵심 필드를 단계 (*) 방식으로 계산한 해시코드이다.
    - 여기서 핵심 필드는 equals 비교에 사용되는 필드를 말한다.
    
- 해당 객체의 나머지 핵심 필드인 f 각각에 대해 다음 작업을 수행한다.
  - 해당 필드의 해시코드 c 를 계산한다. (*)
    - 기본 타입 필드라면, Type.hashCode(f)를 수행한다. 여기서 Type은 해당 기본타입의 박싱 클래스다.
    - 참조 타입 필드면서, 이 클래스의 equals 메소드가 이 필드의 equals를 재귀적으로 호출하여 비교한다면, 이 필드의 hashCode를 재귀적으로 호출한다.
    - 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다.
      - 모든 원소가 핵심 원소라면 Arrays.hashCode를 사용한다.
  - (*) 에서 계산한 해시코드 c로 result를 갱신한다.
    - result = 31 * result + c;
    
- result를 반환한다.
  
#
### 5. hashCode 메서드가 동치인 인스턴스에 대해 같은 해시코드를 반환하는지 확인해 보자!
- 직관을 검증할 단위 테스트!

- 전형적인 hashCode 메서드

```java
@Override
public int hashCode(){
  int result = Short.hashCode(areaCode);
  result = 31 * result + Short.hashCode(prefix);
  result = 31 * result + Short.hashCode(lineNum);
  return result;
}
```
- 동치인 PhoneNumber 인스턴스들은 같은 해시코드를 가질 것이 확실함!
- 근데 무슨 소리인지 잘 모르겠다 (수학 바보 김씨)


#
### 6. Object 클래스가 제공하는 hash 메서드
- Object 클래스는 해시코드를 계산해주는 정적 메서드인 hash를 제공한다 (그럼 이걸 쓰자)
- 단 한줄로도 작성 간으하지만 속도는 더 느리다 (그럼 좀 문제읶군)
- 입력 인수를 받기 위한 배열이 만들어지고 입력중 기본 타입이 있다면 박싱과 언박싱을 거쳐야하기 때문~!
- 그러니 성능에 민간하지 않을 때만 사용해라

```java
@Override
public int hashCode() {
  return Objects.hash(lineNum, prefix, areaCode);
}
```

#
### 7. 그 이외의 방법~!!
- 클래스가 불변이고 해시코드를 계산하는 비용이 크다면 ? -> 캐싱 방식을 고려하라~!!
  - 어떤 타입의 객체가 주로 해시의 키로 사용될 것 같다면 인스턴스가 만들어질 때 해시코드를 계산해둬라

- 혜시의 키로 사용되지 않는 경우라면 hashCode가 처음 불릴 때 계산하는 지연 초기화 전략도 코려해라
  - 필드를 지연 초기화 하려면 그 클래스 스레드를 안전하게 만들도옥 신경 써야 한다(아이템 83가사 이해하자)   

```java
 @Override
 public int hashCode(){
  int result = hashCode;
  if(result==0){
    result = Short.hashCode(areaCode);
    returlt = 31 * result + Short.hashCOde(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    hashCode = result;
  }
  return result;
 }
```

#
### 8. 주의 사항
- 성능 높인다고 해시코드 계산할 때 핵심 필드를 빼먹으면 안된다~!!
- 속도야 빨라지지만 해시 품질이 나빠져 결국에는 성능 끔직해진다.
- 어떤 필드는 해시코드를 넓게 퍼뜨려줄지도 몰라~
- 히필 저런 필드를 생략하면 속도가 선형으로 느려질 거야~!!

- hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세하게 공표하지는 말자~
- 그래야 클라이언트가 이 값에 의지하지 않는다
- String, Integer 바람직 하지 않지만 이미 글렀다~!!









