# 반환 타입으로는 스트림보다 컬렉션이 낫다
원소 시퀀스를 반환하는 메서드에서 반환 타입으로 `Collection`, `Set`, `List`와 같은 컬렉션 인터페이스나 `Iterable`, `배열`이라는 선택지가 있었다.  
기본적으로 컬렉션 인터페이스를 사용하며, for-each문에서만 사용하는 경우에는 Iterable 인터페이스를, 성능에 민감한 경우면 배열을 사용했다.  
그런데 자바 8이 스트림을 지원하면서 부터 선택의 기준이 복잡해지게 되었다.

#
### 스트림 반환

- 스트림은 반복을 지원하지 않기 때문에 for-each문으로 반복하길 원하는 사용자는 불만을 가질 수 있음
> Stream 인터페이스는 Iterable 인터페이스가 정의한 추상 메서드를 전부 포함하고, 정의한 방식대로 동작하지만  
> extend 하지 않기 때문에 스트림을 반복할 수 없다.

#### 스트림을 반복하기 위한 '끔찍한' 우회 방법
```java
for (ProcessHandle ph : (Iterable<ProcessHandle>) ProcessHandle.allProcesses()::iterator) {
  // 프로세스를 처리한다.
}

```
- 작동은 하지만 실전에 사용하기 너무 난잡하고 직관성이 떨어짐

#### Stream<\E>를 Itreable</E>로 중개해주는 어뎁터 사용
```java
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
  return stream::iterator;
}
```
- 에뎁터를 사용하면 어떤 스트림도 for-each문으로 반복 가능



#
### Itreable 반환

#### Itreable<\E>를 Stream</E>로 중개해주는 어뎁터 사용
```java
public static <E> Stream<E> streamOf(Itreable<E> iterable) {
  return StreamSupport.stream(iterable.spliterator(), false);
}
```
- API가 Itreable만 반환해도 스트림 파이프라인에서 처리 가능하게 해주는 어뎁터



#
### Collection 반환
- Collection 인터페이스는 Itreable의 하위 타입이며 stream 메서드도 제공해 반복과 스트림 동시 지원 가능

> 그러므로 원소 시퀀스를 반환하는 공개 API의 반환 타입에는 Collection이나 그 하위 타입을 사용하는 것이 일반적으로 최선!

> 하지만 단지 컬렉션을 반환한다는 이유로 덩치 큰 시퀀스를 메모리에 올려서는 안됨!

#### 전용 컬렉션
- 반환할 시퀀스가 크다면 전용 컬력션을 구현하는 방안 검토하기
- 각 원소의 인텍스를 비트 벡터로 사용
