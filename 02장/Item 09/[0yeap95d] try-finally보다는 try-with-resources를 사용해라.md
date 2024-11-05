### ❓ try-finally의 문제점

**✏ #01 예제소스**

```java
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```

> 예외는 try블록과 finally 블록 모두에서 발생할 수 있다
>
>  ```readLine``` 과 ```close```에서 모두 예외가 발생한다면 ```readLine``` 예외만을 기록



**✏ #02 예제소스**

```java
static void copy(String src, String dst) throws IOException {
	InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.wirte(buf, 0, n);
        } finally {
            out.close();
        }
    } finally {
        in.close();
    }
}
```

> ~~일단 딱봐도 개지저분함~~



#### ```try-finally```를 사용하지 말아야하는 이유

1. ```try-finally```는 자원을 회수하는 최선의 방책이 아니다
2. 위에서 볼 수 있듯이 방식 자체가 너무 지저분하다



---



### 💡 try-with-resources를 사용해라!

- 자바7에서 ```try-with-resources```를 이용하여 위의 문제점을 해결

- 사용하기 위해서 해당 자원이 ```AutoCloseable``` 인터페이스를 구현해야함

  - 단순히 ```void```를 반환하는 ```close ```메서드 하나만 정의한 인터페이스이다

    ~~그렇다면 왜 해당 인터페이스를 구현해야할까?~~



**✏ #01 예제소스 | ```try-with-resource``` 적용**

```java
static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(
    		new FileReader(path))) {
        return br.readLine();
    }
}
```

> ```readLine``` 과 ```close```에서 모두 예외가 발생한다면 ```close``` 에서 발생한 예외는 숨겨지고 ```readLine```에서 발생한 예외가 기록된다
>
> 즉, 프로그래머에게 보여줄 예외 하나만 보존되고 여러 개의 다른 예외가 숨겨질 수 있다

📝 *자바7에서 ```Throwable```에 추가된 ```getSuppressed``` 메서드를 이용하면 프로그램 코드에서 가져올 수도 있다*



**✏ #02 예제소스 | ```try-with-resource``` 적용**

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



**✏ #03 예제소스 | ```try-with-resource & catch``` **

```java
static String firstLineOfFile(String path, String defaultVal) {
    try (BufferedReader br = new BufferedReader(
    		new FileReader(path))) {
        return br.readLine();
    } catch (IOException e) {
        return defaultVal;
    }
}
```

> ```catch```절을 이용하여 ```try```문을 중첩하지 않고 다수의 예외를 처리
>
> 예외를 던지는 대신 기본값을 반환 



#### ```try-with-resources```를 사용해야하는 이유

1. 자원을 회수하는 최선책이다
2. 짧고 읽기 수월할 뿐 아니라 문제를 진단하기도 훨씬 좋다



---



### 📌 핵심정리

**꼭 회수해야하는 자원이라면 ```try-finall``` 말고, ```try-with-resources```를 사용하자!!** 

~~그냥 try-with-resources가 짱임 암튼 그럼~~
