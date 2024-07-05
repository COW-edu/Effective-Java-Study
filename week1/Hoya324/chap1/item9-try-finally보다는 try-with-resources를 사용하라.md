# item9-try-finally보다는 try-with-resources를 사용하라

## 핵심

꼭 회수해야 하는 자원을 다룰 때는 try-finally 말고, try-with-resources를 사용하자.

예외는 없다. 코드는 더 짧고 분명해지고, 만들어지는 예외 정보도 훨씬 유용하다.

try-finally로 작성하면 실용적이지 못할 만큼 코드가 지저분해지는 경우라도, try-with-resources로는 정확하고 쉽게 자원을 회수할 수 있다.

## 이유

### close를 여러번 호출하는 경우 try-finally의 문제점

만약 close를 여러 번 호출해야 하는 상황이 오면 어떻게 될까?

```java
package effectivejava.chapter2.item9.tryfinally;

import java.io.*;

public class Copy {
    private static final int BUFFER_SIZE = 8 * 1024;

    // 코드 9-2 자원이 둘 이상이면 try-finally 방식은 너무 지저분하다! (47쪽)
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

    public static void main(String[] args) throws IOException {
        String src = args[0];
        String dst = args[1];
        copy(src, dst);
    }
}
```

- 코드가 굉장히 지저분해지게 될 수 있다.

```java
package effectivejava.chapter2.item9.tryfinally;

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;

public class TopLine {
    // 코드 9-1 try-finally - 더 이상 자원을 회수하는 최선의 방책이 아니다! (47쪽)
    static String firstLineOfFile(String path) throws IOException {
        BufferedReader br = new BufferedReader(new FileReader(path));
        try {
            return br.readLine();
        } finally {
            br.close();
        }
    }

    public static void main(String[] args) throws IOException {
        String path = args[0];
        System.out.println(firstLineOfFile(path));
    }
}
```

- 또한 더 큰 문제점은 예외는 try블록과 finally 블록 모두에서 발생할 수 있는데, 예상치 못한 문제로 firstLineOfFile 메서드 안의 readLine 메서드가 예외를 던지고, 같은 이유로 colse 메서드도 완전히 실패할 것이다.
- 이런 상황이라면 두 번째 에외가 첫 번째 예외를 완전히 집어삼켜 버린다. 그러면 스택 추적 내역에 첫 번째 예외에 관한 정보는 남지 않게 되어, 실제 시스템에서의 디버깅을 몹시 어렵게 한다.

### 자원을 회수하는 최선책 - try-with-resources

이러한 try-finally 방식의 단점을 보완하기 위해 자바 7 버전부터는 try-with-resources가 도입되었다. try-with-resources를 사용하기 위해서는 사용하는 자원이 AutoCloseable 인터페이스를 구현해야 한다.

```java
public interface AutoCloseable {

    void close() throws Exception;
}
```

AutoCloseable 인터페이스는 단지 close 메서드 하나만을 정의해 놓은 간단한 인터페이스이며, 자바 라이브러리와 서드파티 라이브러리의 수많은 클래스와 인터페이스는 이미 AutoCloseable을 구현하거나 확장해두었다.

따라서, close가 필요한 자원 클래스를 커스텀 할 일이 있다면 AutoCloseable을 반드시 구현할 것을 권장한다.

- try-with-resources 구조 역시 기존 try-finally 처럼 catch를 병용해서 사용할 수 있다.

```java
public static String firstLineOfFile(String path) {
    try (BufferedReader br = new BufferedReader(new InputStream(System.in))) {
        return br.readLine();
    } catch (IOException e) {
        return "IOException 발생";
    }
}
```

이렇게 숨겨진 예외는 무시되는 것이 아니라, suppressed 상태가 되어 stackTrace 시 숨겨졌다는 메시지로 출력된다. suppressed 상태가 된 예외는 자바 7부터 도입된 getSuppressed 메서드를 통해 가져와서 사용할 수 있다.
