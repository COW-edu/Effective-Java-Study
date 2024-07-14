# item7-다 쓴 객체 참조를 해제하라

## 핵심

메모리 누수는 겉으로 잘 드러나지 않아 시스템에 수년간 잠복하는 사례도 있다. 

이런 누수는 철저한 코드 리뷰나 힙 프로파일러 같은 디버깅 도구를 동원해야만 발견되기도 한다. 

그래서 이런 종류의 문제는 예방법을 익혀두는 것이 매우 중요하다.

## 이유

### 메모리 누수가 일어나는 위치는 어디인가?

### 1. 메모리를 직접 관리하였으나 제거하지 않은 경우

```java
package effectivejava.chapter2.item7;
import java.util.*;

// 코드 7-1 메모리 누수가 일어나는 위치는 어디인가? (36쪽)
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }

    /**
     * 원소를 위한 공간을 적어도 하나 이상 확보한다.
     * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
     */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }

    public static void main(String[] args) {
        Stack stack = new Stack();
        for (String arg : args)
            stack.push(arg);

        while (true)
            System.err.println(stack.pop());
    }
}
```

- **스택이 커졌다가 줄어들 때 스택에서 꺼내진 객체들을 가비지 컬렉터가 회수하지 않는다.**
- 객체를 더이상 사용하지 않더라도, 객체들의 다 쓴 참조(obsolete reference)를 여전히 가지고 있기 때문이다.
- 이 스택을 사용하는 프로그램을 오래 실행하다보면 가비지 컬렉션 활동과 메모리 사용량이 늘어나 결국 성능이 저하될 것이다.
- 심한 경우 디스크 페이징이나 OutOfMemoryError를 일으켜 프로그램이 예기치 않게 종료되기도 한다.
- 이는 스택이 자기 메모리를 직접 관리하기 때문에 이런 것이다.
- 즉, 가비지 컬렉터가 보기엔 활성, 비활성 영역이 똑같이 유효한 객체로 보이기 때문이다.

**해결방법은 간단하게 null을 넣어주면 된다.**

```java
// 코드 7-2 제대로 구현한 pop 메서드 (37쪽)
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; // 다 쓴 참조 해제
    return result;
}
```

### 2. 캐시 역시 메모리 누수의 주범이다.

**캐시 방법에는 여러가지 방법이 있다.**

- WeakHashMap
    - 캐시 외부에서 key를 참조하는 동안만 엔트리가 살아있는 캐시가 필요한 상황이라면 WeakHashMap을 사용해 캐시를 만들자
    - 이러면 다 쓴 엔트리는 즉시 자동 제거될 것이다.
- TreadPoolExecutor 같은 백그라운드 스레드를 활용하거나 캐시에 새 엔트리를 추가할 때 부수 작업으로 수행하는 방법
- java.lang.ref 패키지를 직업 활용하는 방법

### 3. 리스너(listener) 또는 콜백(callback)

- 클라이언트가 콜백만 등록하고 명확히 해지하지 않는다면 콜백이 쌓여만 간다.
- 이때 콜백을 WeakHashMap과 같은 약한 참조(weak reference)에 저장하면 가비지 컬렉터가 즉시 수거한다.

## 고려 사항

### Null 처리는 예외적인 경우여야 한다.

- 가장 좋은 방법은 그 참조를 담은 변수를 유효 범위 밖으로 밀어내는 것이다.
- **item57에서 범위를 최소로 정의하는 방법을 다루면 자연스럽게 습득할 것**
