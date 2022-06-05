## 1. 들어가기

clone 메서드를 직접 사용해본 경험이 있나요?

Object를 상속 받는 모든 클래스는 clone 메서드를 호출할 수 있습니다.

하지만, 저는 clone 메서드를 호출해 본 경험은 없네요...

그래서 먼저, clone 메서드가 어떤 역할을 하는지 알아보도록 하겠습니다.

## 2. clone 메서드

clone 메서드는 원본 객체의 필드값과 동일한 값을 가지는 새로운 객체를 생성합니다.

즉, 여러 정보를 담은 객체를 복제해서 그 객체와 똑같은 정보를 가진 또 하나의 객체를 생성하는 것입니다.

하지만, 모든 클래스가 이 clone 메서드를 호출할 수 있는 것은 아닙니다.

바로, Cloneable이라는 인터페이스가 구현된 클래스여야 호출 가능합니다.

Cloneable 인터페이스가 구현되지 않았는데 clone 메서드를 호출하게 되면

CloneNotSupportedException 예외가 발생합니다.

```java
/* CloneTest 클래스 */
		public class CloneTest {
				@Override
				protected Object clone() throws CloneNotSupportedException {
						return super.clone();
				}
		}

/* Client */
		CloneTest test1 = new CloneTest();
		CloneTest test2 = (CloneTest) test1.clone();
```
```
		Exception in thread "main" java.lang.CloneNotSupportedException: CloneTest
```

## 3. Cloneable 인터페이스

그럼 Cloneable 인터페이스는 어떤 역할을 하는걸까요?

어떤 역할을 하는지 알아보기 위해 Cloneable 인터페이스의 내부를 살펴보겠습니다.

```java
		package java.lang;

		public interface Cloneable {
		}
```

놀랍게도 Cloneable 인터페이스의 내부는 아무것도 없습니다.

보통 일반적인 인터페이스는 구현 클래스에서 미리 구현할 기능을 제공합니다.

그럼 내부에 아무것도 없는 Cloneable 인터페이스는 도대체 어떤 역할을 하는걸까요?

Cloneable 인터페이스는 Object의 protected 메서드인 clone의 동작 방식을 결정하는 역할을 합니다.

실무에서 Cloneable을 구현한 클래스는 clone 메서드를 public으로 제공하며,

사용자는 당연히 복제가 제대로 이뤄지리라 기대합니다.

이 기대를 만족시키기 위해서는 그 클래스와 모든 상위 클래스는 복잡하고,

강제할 수 없고, 허술하게 기술된 프로토콜을 지켜야 하는데

그 결과, 깨지기 쉽고, 위험하고, 모순적인 매커니즘이 탄생합니다.

즉, 생성자를 호출하지 않고도 객체를 생성할 수 있게 되는 것입니다.

## 4. clone 메서드의 일반 규약

Object 명세의 clone 메서드 일반 규약을 알아봅시다.

* ``` x.clone() != x ```

* ``` x.clone().getClass() == x.getClass() ```

* ``` x.clone().equals(x) ```

이상의 규약은 참이지만, 반드시 만족해야 하는 것은 아닙니다.

그리고 관례상, 반환된 객체와 원본 객체는 독립적이어야 합니다.

이를 만족하기 위해서는 super.clone으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정할 수도 있습니다.

## 5. clone 메서드 작성법

그럼, 제대로 동작하는 clone 메서드를 한 번 만들어볼까요?

그 전에 상위 클래스에서 이미 제대로 동작하는 clone 메서드를 가지고 있다고 가정해보겠습니다.

1. super.clone 호출

	먼저, super.clone 메서드를 통해 객체를 복제합니다.

	이렇게 복제된 객체는 원본 필드와 똑같은 값을 가지는 완벽한 복제본일 것입니다.

2. 필드 타입을 확인합니다.

	* 모든 타입이 기본 타입이거나 불변 객체를 참조하는 경우

	```java
		public class Book implements Cloneable {
			String name;
			String writer;

			@Override
			public Book clone() {
				try {
					return (Book) super.clone();
				} catch (CloneNotSupportedException e) {
					throw new AssertionError();
				}
			}
		}
	```

	모든 타입이 기본 타입이거나 불변 객체를 참조하는 경우에는 따로 처리해줄 것은 없습니다.

	코드만 간단하게 살펴보겠습니다.

	Object의 clone 메서드는 Object를 반환하지만 Book 클래스의 clone 메서드는 Book을 반환합니다.

	이는 Java가 공변 반환 타이핑(Covariant Return Typing)을 지원하기 때문에 가능합니다.

	> * 공변 반환 타이핑 (Covariant Return Typing)
	>
	> 	메서드가 재정의될 때 더 좁은 타입으로 교체 가능

	공변 반환 타이핑을 사용하면 클라이언트에서 형변환을 하지 않아도 되는 장점이 있습니다.

	또한, clone 메서드를 자세히 살펴보면 try-catch 문을 사용한 것을 볼 수 있습니다.

	그 이유는 Object의 clone 메서드가 checked Exception을 던지도록 선언되었기 때문입니다.

	> * 검사 예외 (Checked Exception)
	> 
	> 	반드시 예외처리 해야 하는 예외

	* 가변 객체를 참조하는 경우

	```java
		public class Stack implements Cloneable {
			private Object[] elements;

			@Override
			public Stack clone() {
				try {
					Stack result = (Stack) super.clone();
					result.elements = elements.clone();
					return result;
				} catch (CloneNotSupportedException e) {
					throw new AssertionError();
				}
			}
		}
	```

	클래스가 가변 객체를 참조하는 경우에는 복잡해질 수 있습니다.

	> * 가변 객체
	>
	> 	Class의 인스턴스가 생성된 이후에 내부 상태가 변경 가능한 객체
	>
	>	ex)	Array, ArrayList, HashMap, StringBuilder 등

	Item 7에서 사용했던 Stack 클래스를 예시로 들어보겠습니다.

	만약, 기본 타입이나 불변 타입 예시와 같이 super.clone()을 그대로 반환하면 어떻게 될까요?

	겉으로 보기에는 원본과 동일한 객체를 복제했다고 생각할 수 있습니다.

	하지만, 원본 elements 값을 변경할 경우, 복제 객체 elements도 동일하게 값이 변경됩니다.

	그렇기 때문에 제대로 복제하기 위해서는 내부 정보를 복사해야 하는데

	가장 쉬운 방법은 ``` result.elements = elements.clone(); ``` 처럼 clone을 재귀적으로 호출하는 것입니다.

	배열의 clone은 원본 배열과 똑같은 타입의 배열을 반환하기 때문에 형변환할 필요는 없습니다.

	한편, elements 필드가 final이었다면 앞서의 방식은 작동하지 않습니다.

	final 필드에는 새로운 값을 할당할 수 없기 때문입니다.

	그래서 복제할 수 있는 클래스를 만들기 위해 final 한정자를 제거해야할 수도 있습니다.

	* 복잡한 가변 상태를 참조하는 경우

	그러나 때론, 단순한 재귀 호출만으로 충분하지 않을 때도 있습니다.

	```java
		public class HashTable implements Cloneable {
			private Entry[] buckets;

			private static class Entry {
				final Object key;
				Object value;
				Entry next;

				Entry(Object key, Object value, Entry next) {
					this.key = key;
					this.value = value;
					this.next = next;
				}
			}
		}
	```

	해당 코드는 해시테이블 클래스이고 해시테이블의 내부는 버킷들의 배열입니다.

	그리고 각 버킷은 키-값 쌍을 담는 연결 리스트의 첫 번째 엔트리를 참조합니다.

	단순하게 여기서 clone 메서드를 재귀적으로 호출해보겠습니다.

	```java
		@Override
		public HashTable clone() {
			try {
				HashTable result = (HashTable) super.clone();
				result.buckets = buckets.clone();
				return result;
			} catch (CloneNotSupportedException e) {
				throw new AssertionError();
			}
		}
	```

	복제 객체는 자신만의 버킷 배열을 가집니다.

	하지만 이 배열은 원본 객체와 같은 연결 리스트를 참조하기 때문에

	원본 객체와 복제 객체 모두 예기치 않게 동작할 가능성이 생깁니다.

	이를 해결하기 위해서는 각 버킷을 구성하는 연결 리스트도 복사해야 합니다.

	그럼 다시 clone 메서드를 수정해보겠습니다.

	```java
		public class HashTable implements Cloneable {
			private Entry[] buckets;

			private static class Entry {
				final Object key;
				Object value;
				Entry next;

				Entry(Object key, Object value, Entry next) {
					this.key = key;
					this.value = value;
					this.next = next;
				}

				// 1. 연결리스트를 재귀적으로 복사
				Entry deepCopy() {
					return new Entry(key, value, next == null ? null : next.deepCopy());
				}
			}

			@Override
			public HashTable clone() {
				try {
					HashTable result = (HashTable) super.clone();

					// 2. 각 버킷을 deepCopy 메서드를 통해 연결리스트 재귀적으로 복사
					result.buckets = new Entry[buckets.length];
					for(int i=0; i<buckets.length; i++) {
						if(buckets[i] != null)
							result.buckets[i] = buckets[i].deepCopy();
					}

					return result;
				} catch (CloneNotSupportedException e) {
					throw new AssertionError();
				}
			}
		}
	```

	먼저 적절한 크기의 새로운 버킷 배열을 할당합니다.

	그 후, 원래 버킷 배열을 순회하면서 비지 않은 각 버킷에 대해 deepCopy를 수행합니다.

	deepCopy 메서드는 자신이 가리키는 연결 리스트 전체를 복사하기 위해 자신을 재귀 호출합니다.

	이 방법은 간단하고 버킷이 너무 길지만 않으면 잘 작동합니다.

	하지만, 버킷이 너무 긴 경우에는 Stack Overflow를 일으킬 위험이 있습니다.

	그 때는 재귀 호출 대신 반복자 순회 방법을 사용할 수 있습니다.

	```java
		Entry deepCopy() {
			Entry result = new Entry(key, value, next);
			
			for(Entry p = result; p.next != null; p=p.next)
				p.next = new Entry(p.next.key, p.next.value, p.next.next);

			return result;
		}
	```

	재귀, 반복 순회 외에도 고수준 메서드를 사용하는 방법이 있습니다.

	하지만, 고수준 메서드를 사용하면 간단하고 우아한 코드를 얻을 수 있지만,

	저수준 메서드에서 바로 처리할 때보다 느리고 전체 Cloneable 아키텍처와 어울리지 않는 방식입니다.

## 6. clone 메서드 작성 주의사항

이번에는 clone 메서드 작성 시에 주의해야할 사항을 알아보겠습니다.

1. 재정의될 수 있는 메서드를 호출하지 않아야 한다.

	clone 메서드에서 하위 클래스가 재정의한 메서드를 호출하게 되면,

	하위 클래스는 복제 과정에서 자신의 상태를 교정할 기회를 잃게 됩니다.

	그래서 이는 원본 객체와 복제 객체의 상태가 달라질 가능성이 큽니다.

2. throws 절을 제거해야 한다.

	만약, clone 메서드에 throws 절이 포함되어 있다면

	클라이언트 코드에서 clone 메서드를 사용할 때마다 예외처리를 해야 합니다.

	그렇게 된다면 코드의 가독성을 해치고 개발자가 매우 귀찮아 할 수 있습니다.

3. 상속용 클래스는 Cloneable을 구현해서는 안된다.

	제대로 작동하는 clone 메서드를 구현해 protected로 두고

	CloneNotSupportedException도 던질 수 있다고 선언하는 것이기 때문입니다.
	
	다른 방법으로는 clone 메서드를 다음과 같이 퇴화시키는 방법이 있습니다.

	```java
		@Override
		protected final Object clone() throws CloneNotSupportedException {
			throw new CloneNotSupportedException();
		}
	```

	4. Thread-safe 클래스를 작성할 때에는 clone 메서드도 동기화 해줘야 한다.

	Object clone 메서드는 동기화를 신경 쓰지 않았기 때문에

	MultiThread 환경에서 재정의해서 사용하는 clone 메서드는 따로 동기화를 해줘야 합니다.

## 7. 또다른 객체 복제 방식

객체를 복제하는 방법은 clone 메서드를 사용하는 방법 뿐일까요?

clone 메서드보다 더 나은 두 가지의 객체 복사 방식을 한 번 알아보겠습니다.

1. 복사 생성자

	```java
		public Yum(Yum yum) {}
	```

2. 복사 팩터리

	```java
		public static Yum newInstance(Yum yum) {}
	```

이 두 가지 방식은 아래와 같이 clone 메서드의 불편하고 위험한 요소로부터 벗어날 수 있습니다.

* 언어 모순적이다.

* 생성자를 쓰지 않고 객체를 생성하는 위험한 객체 생성 메커니즘

* final 필드와의 충돌

* 불필요한 checked Exception

* 형변환 필요

추가로, 두 가지 방식은 해당 클래스가 구현한 인터페이스 타입의 인스턴스를 인수로 받을 수 있습니다.

예를 들어 대부분의 범용 Collection 구현체는 Collection이나 map 타입을 받는 생성자를 제공합니다.

이들을 이용하면 클라이언트는 원본의 구현 타입에 얽매이지 않고 복제본의 타입을 직접 선택할 수 있습니다.

## 8. 정리

이번 포스트에서는 clone 메서드가 무엇이고, 작성법과 주의사항에 대해 알아보았습니다.

Cloneable이 몰고 온 문제를 되짚어봤을 때,

새로운 인터페이스를 만들 때는 절대 Cloneable을 확장해서도 새로운 클래스도 이를 구현해서는 안됩니다.

clone 메서드는 객체 필드의 타입이 무엇이냐에 따라 원본 객체가 제대로 동작하지 않는 위험이 있기 때문에

배열을 복제할 때에 적합하며, 그 이외를 복제할 때는 생성자와 팩터리를 이용하는게 좋습니다.