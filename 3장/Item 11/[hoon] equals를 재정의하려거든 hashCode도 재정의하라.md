## 1. 들어가기

hashCode란 무엇일까요?

대부분 이런 질문을 받으면 '어...' 가 먼저 나올 것입니다.

저 또한 들어봤지만 자세한 설명은 하지 못하겠네요...

그럼 먼저, hashCode가 무엇인지 알아봅시다.

## 2. hashCode

hashCode란 객체를 식별하는 하나의 정수 값을 의미합니다.

Object의 hashCode() 메서드는 객체의 메모리 번지를 이용해 hashCode를 만들기 때문에

각 객체마다 다른 값을 가지고 있습니다.

Object 명세에서 정의한 hashCode 규약을 보겠습니다.

> * equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안
>
>   그 객체의 hashCode 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다.
>
>   단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관없다.

> * equals(Object)가 두 객체를 같다고 판단했다면,
>
>   두 객체의 hashCode는 똑같은 값을 반환해야 한다.

> * equals(Object)가 두 객체를 다르다고 판단했더라도,
>
>   두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다.

이는 hashCode를 재정의하더라도 지켜야합니다.

위의 규약 중 hashCode 재정의를 잘못했을 경우, 크게 문제가 되는 조항은 두 번째입니다.

예시를 통해 알아보겠습니다.

```java
/* Key 클래스 */
  class Key {
    private int num;

    Key(int num) {
      this.num = num;
    }

    @Override
    public boolean equals(Object obj) {
      if(obj instanceof Key) {
        Key compareKey = (Key) obj;

        if(this.num == compareKey.num)
          return true;
      }

      return false;
    }
  }

/* Client */
  public static void main(String[] args) {
    HashMap<Key, String> map = new HashMap<>();

    map.put(new Key(1), "abc");

    String ans = map.get(new Key(1));
    System.out.println(ans);
  }
```
```
  null
```

하나의 숫자 필드를 가진 Key 클래스는 같은 필드 값을 가지면

동일한 객체로 판단되도록 equals 메서드를 재정의 했습니다.

하지만 equals 메서드만 재정의하고 hashCode 메서드는 재정의하지 않았습니다.

그랬더니 필드 값이 같은 객체를 hashMap에서 찾으려 해도 찾을 수 없는 문제점이 발생합니다.

그 이유는 두 객체는 현재 논리적으로는 같지만, hashCode 값이 다르기 때문입니다.

그렇다면 아래와 같이 hashCode 메서드를 재정의하면 어떨까요?

```java
  @Override
  public int hashCode() {
    return 1;
  }
```

```
  abc
```

논리적으로 같고, 같은 hashCode 값을 가지기 때문에

'abc' 라는 결과가 나오게 됩니다.

하지만, 이럴 경우 모든 객체가 hashCode 값을 1로 가지기 때문에

hashTable이 마치 연결리스트처럼 동작해서

평균 수행 시간이 O(1)인 hashTable이 O(n)으로 느려지게 됩니다.

<div class="div-post-img">
  <img src="https://ss-hoon.github.io/assets/img/effective_java/item11/hashtable-collision.png" width="40%" height="30%" />
</div>

그렇다면 어떻게 hashCode를 작성하면 될까요?

## 3. 좋은 hashCode 작성 요령

1. int 변수 result를 선언한 후, 값 c로 초기화한다.

   ( c는 해당 객체의 첫 번째 핵심 필드를 단계 2.1 방식으로 계산 )

2. 해당 객체의 나머지 핵심 필드 f 각각에 대해 다음 작업을 수행한다.

    1. 해당 필드의 해시코드 c를 계산한다.

        1. 기본 타입 필드라면 Type.hashCode(f)를 수행한다. (Type은 해당 기본 타입의 박싱 클래스)

        2. 참조 타입 필드면서 해당 필드의 equals를 재귀적으로 호출해 비교한다면,

           이 필드의 hashCode를 재귀적으로 호출한다.

           계산이 복잡해질 것 같으면 표준형을 만들어서 hashCode를 호출한다.

        3. 필드가 배열이라면 핵심 원소 각각을 별도 필드처럼 다룬다.

           모든 원소가 핵심 원소라면 Arrays.hashCode를 사용한다.

    2. 단계 2.1에서 계산한 hashCode c로 result를 갱신한다.

       ``` result = 31 * result + c ```

3. result를 반환한다.

다음은 hashCode 작성 요령을 참조한 예시입니다.

```java
  @Override
  public int hashCode() {

    // 핵심 필드 (1)
    int result = Short.hashCode(firstField);

    // 기본 타입 필드 (2.1.1)
    result = 31 * result + Integer.hashCode(secondField);

    // 참조 타입 필드 (2.1.2)
    result = 31 * result + (thirdField == null ? 0 : thirdCode.hashCode());

    // 배열 필드 (2.1.3)
    for(Item element : fourthField) {
      result = 31 * result + (element == null ? 0 : element.hashCode());
    }

    return result;
  }
```

## 4. hashCode를 재정의할 때 주의할 점

1. 성능을 높인답시고 hashCode를 계산할 때 핵심 필드를 생략해서는 안된다.

2. hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말자.

   그래야 클라이언트가 이 값에 의지하지 않게 되고, 추후에 계산 방식을 바꿀 수도 있다.

## 5. hashCode를 얻는 다른 방법

지금까지 hashCode를 직접 재정의하는 방법에 대해 알아보았습니다.

하지만, hashCode를 이미 만들어진 메서드를 호출해서 사용하는 방법도 있습니다.

1. Guava의 com.google.common.hash.Hashing

2. Objects 클래스의 hash 메서드

```java
  @Override
  public int hashCode() {
    return Objects.hash(firstField, secondField, thirdField);
  }
```

## 6. 정리

이번 포스트에서는 hashCode에 대해서 알아보았습니다.

hashCode는 객체를 식별하는 하나의 정수 값이자, hashTable의 key 값입니다.

객체를 식별하는 hashCode이기에 equals 메서드만 재정의하고 hashCode 메서드는 재정의하지 않을 경우,

hashMap이나 hashSet과 같은 hashCode를 사용하는 컬렉션에서

해당 객체를 찾으려해도 찾을 수 없는 문제가 발생합니다.

그렇기 때문에 equals 메서드를 재정의할 때에는 반드시 hashCode 메서드도 함께 재정의해야 합니다.

## 7. 참조

* [https://en.wikipedia.org/wiki/Hash_function](https://en.wikipedia.org/wiki/Hash_function)

* [https://100100e.tistory.com/352](https://100100e.tistory.com/352)

* [https://k3068.tistory.com/m/63](https://k3068.tistory.com/m/63)
