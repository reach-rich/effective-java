### 📝 표준 예외 재사용

- API가 다른 사람이 익히고 사용하기 쉬워짐
- 낯선 예외를 사용하지 않게 되어 읽기 쉬움
- 예외 클래스 수가 적을수록 메모리 사용량도 줄고 클래스 적재하는 시간도 적음

| 예외                            | 주요쓰임                                                     |
| ------------------------------- | ------------------------------------------------------------ |
| IllegalArgumentException        | 허용하지 않는 값이 인수로 건네졌을 때(null은 따로 NullPointerException으로 처리) |
| IllegalStateException           | 객체가 메서드를 수행하기에 적절하지 않은 상태일 때           |
| NullPointerException            | null을 허용하지 않는 메서드에 null을 건넸을 때               |
| IndexOutOfBoundsException       | 인덱스가 범위를 넘어섰을 때                                  |
| ConcurrentModificationException | 허용하지 않는 동시 수정이 발견됐을 때                        |
| UnsupportedOperationException   | 호출한 메서드를 지원하지 않을 때                             |

<br>

**✔Exception, RuntimeException, Throwable, Error는 직접 재사용하지 말자**

>이 예외들은 다른 예외들의 상위 클래스, 즉여 러 성격의 예외들을 포괄하는 클래스이므로 안정적으로 테스트할 수 없음
