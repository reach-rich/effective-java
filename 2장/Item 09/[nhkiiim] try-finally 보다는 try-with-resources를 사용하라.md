# try-finally 보다는 try-with-resources를 사용하라
### 1. 자바 라이브러리에는 close 메서드를 호출해 직접 닫아줘야 하는 자원이 많움

- InputStream, OutputStream, java.sql.Connection 같은 칭구들

- 자원 닫기는 클라이언트가 놓치기 쉬워서 예측할 수 없는 성능 문제로 이어지기도 함
- 전통적으로 자원이 제대로 닫힘을 보장하는 수단으로 try-finally를 씀

 

#
### 2.try-finally
- try-finally - 더 이상 자원을 회수하는 최선의 방책이 아녀유!!
```java
public static String firstLineOfFile(String path) throw IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```

- 자원이 둘 이상이면 try-finally 방식은 너무 지저분!
```java
static void copy(String src, String dst) throws IOException {
	InputStream in = new FileInputStream(src);
	try {
		OutputStream out = new FileOutputStream(dst);
		try {
			byte[] buf = new byte[BUFFER_SIZE];
			int n;
			while ((n = in.read(buf)) >= 0)
				out.write(buf, 0, n);
		} finally {
			out.close();
		}
	} finally {
		in.close();
	}
}
```

- 앞의 두 코드 예제에는 결점 존재 -> 예외는 try 블록과 finally 블록 모두에서 발생 가능 (자원을 해제하는 close 메서드를 호출할 때)
- finally 블록 내에서 다시 try-finally 를 사용하면 코드는 길어지고 가독성도 떨어짐

 

#
### 3. try-with-resources
- 위에 문제를 해결하려고 자바 7에서 투척~! 한 try-with-resources
- 이 구조를 사용하려면 해당 자원이 AutoCloseable 인터페이스를 구현해야함

- AutoCloseable: 단순히 void를 반환하는 close 메서드 하나만 정의한 인터페이스
- 자바의 라이브러리와 서드파티 라이브러리들의 수많은 클래스가 이미 구현해둠
- 닫아야 하는 자원을 사용하면 꼭 구현해줘라!

<br>

- 재작성한 코드들
1)
```java
public static String firstLineOfFile(String path) throw IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    }
}
```

2)
```java
static void copy(String src, String dst) throws IOException {
	try (InputStream in = new FileInputStream(src);
		OutputStream out = new FileOutputStream(dst)) {
		byte[] buf = new byte[BUFFER_SIZE];
		int n;
		while ((n = in.read(buf)) >= 0)
			out.write(buf, 0, n);
	}
}
```

- 이 친구들이 훨씬 읽기 수월하고 문제 진단에도 효율적!
- 1)을 보면 readLine과 close 양쪽 모두에서 예외가 발생해도 close거는 숨겨지고 readLine거만 기록됨
- 프러그래머에게 보여둘 예외만 보존하고 다른 예외는 숨겨질 수 있음
- 숨겨졌다는 꼬리표 (suppressed)를 달고 출력되기는 함
- Throwable에 추가된 getSuppressed 메서드를 이용하면 코드에서 가져오기도 가능

<br>

- catch 절 사용도 가능해서 try문을 중첩하지 않아도 된다는 장점도 있음

```java
static String firstLineOfFile(String path, String defaultVal){
  try(BufferedReafer br = new BufferedReader(
    new FileReader(path))){
      return br.readLine();
  }catch(IOException e){
    return defaultVal;
  }
}
```

#
### 그래서!
- 꼭 회수해야하는 자원을 다룰때는 try-with-resources를 사용하자

- 예외는 없다! 무적권이다!

