## 과도한 동기화는 피하라

> 과도한 동기화는 성능을 떨어뜨리고, 교착상태에 빠뜨리고, 심지어 예측할 수 없는 동작을 낳기도 한다.


### 👽 외계인 메서드
- 응답 불가와 안전 실패를 피하려면 동기화 메서드나 동기화 블록 안에서는 제어를 절대로 클라이언트에 양도하면 안됨
- 동기화된 영역 안에서는 재정의할 수 있는 메서드는 호출 X
- 클라이언트가 넘겨준 함수 객체를 호출하는 것도 X
- 메서드가 무슨 일을 할지 알지 못하며 통제할 수 없는 위와같은 메서드들은 외계인 메서드라고 한다.

> 외계인 메서드가 하는 일에 따라 예외를 발생 시키거나 교착상태에 빠지거나 데이터를 훼손할 수 있음


```java
public class ObservableSet<E> extends ForwardingSet<E> {

    public ObservableSet(Set<E> set) {
        super(set);
    }

    private final List<SetObserver<E>> observers = new ArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        synchronized (observers) {
            observers.add(observer);
        }
    }
    
    public boolean removeObserver(SetObserver<E> observer) {
        synchronized (observers) {
            return observers.remove(observer);
        }
    }
    
    private void notifyElementAdded(E element) {
        synchronized (observers) {
            for(SetObserver<E> observer : observers) {
                observer.added(this, element);
            }
        }
    }
    
    @Override
    public boolean add(E element) {
        boolean added = super.add(element);
        if(added) {
            notifyElementAdded(element);
        }
        return added;
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c) {
            result |= add(element); //notifyElementAdded를 호출
        }
        return result;
    }
}
```
- 관찰자들은 addObserver와 removeObserver 메서드를 호출해 구독을 신청하거나 해지

#
### 👽 예시 1 : 자기 자신을 구독해지하는 관찰자
> 0부터 99까지를 출력하는 프로그램 > 평상시에는 앞서와 같이 집합에 추가된 정숫값을 출력하다가, 그 값이 23이면 자기 자신을 제거하는 관찰자를 추가
```java
public static void main(String[] args) {
	ObservableSet<Integer> set = new ObservableSet<>(New HashSet<>());
	
	set.addObserver(new SetObserver<Integer>() {
		public void added(ObservableSet<Integer> s, Integer e) {
			System.out.println(e);
			if (e == 23) s.removeObserver(this);
		}
	});

	for (int i = 0; i < 100; i++) 
		set.add(i);
}
```
- 이 프로그램은 23까지 출력한 다음 ConcurrentModificationException을 던짐
- added 메서드 호출이 일어난 시점이 notifyElementAdded가 관찰자들의 리스트를 순회하는 도중이기 때문
- added 메서드는 ObservableSet의 removeObserver 메서드를 호출하고, 이 메서드는 다시 observers.remove 메서드를 호출하여 문제 발생

#
### 👽 예시 2 : 다른 스레드에게 구독해지를 부탁하는 관찰자
> removeObserver를 직접 호출하지 않고 실행자 서비스를 사용해 다른 스레드에게 부탁하는 관찰자
  
