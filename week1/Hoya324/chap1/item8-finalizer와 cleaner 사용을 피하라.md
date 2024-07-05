# item8-finalizer와 cleaner 사용을 피하라

## 핵심

cleaner(java 8까지는 finalizer)는 안전망 역할이나 중요하지 않은 네이티브 자원 회수용으로만 사용하자.

물론 이런 경우라도 불확실성과 성능 저하에 주의해야한다.

## 이유

### finalizer는 예측할 수 없고, 상황에 따라 위험할 수 있다.

- finalizers는 java의 객체 소멸자

### cleaner는 finalizer보다는 덜 위험하지만, 여전히 예측할 수 없고, 느리고, 일반적으로 불필요하다.

### finalizer과 cleaner로는 제때 실행되어야하는 작업을 할 수 없다.

- 객체에 접근 할 수 없게 된 후 실행까지 얼마나 걸릴지 알 수 없다.

### 상태를 영구적으로 수정하는 작업에서는 절대 finalizer나 cleaner에 의존해서는 안 된다.

- DB에서 영구 락 해제를 finalizer나 cleaner에 맡기면 분산 시스템 전체가 서서히 멈출 것이다.

### 심각한 성능 문제 동반

### finalizer를 사용한 클래스는 finalizer 공격에 노출되어 심각한 보안 문제를 일으킬 수도 있다.

## 관련 코드 및 예시

`Adult.class`

```java
package effectivejava.chapter2.item8;

// cleaner 안전망을 갖춘 자원을 제대로 활용하는 클라이언트 (45쪽)
public class Adult {
    public static void main(String[] args) {
        try (Room myRoom = new Room(7)) {
            System.out.println("안녕~");
        }
    }
}
```

`Room.class`

```java
package effectivejava.chapter2.item8;

import java.lang.ref.Cleaner;

// 코드 8-1 cleaner를 안전망으로 활용하는 AutoCloseable 클래스 (44쪽)
public class Room implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();

    // 청소가 필요한 자원. 절대 Room을 참조해서는 안 된다!
    private static class State implements Runnable {
        int numJunkPiles; // Number of junk piles in this room

        State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }

        // close 메서드나 cleaner가 호출한다.
        @Override public void run() {
            System.out.println("Cleaning room");
            numJunkPiles = 0;
        }
    }

    // 방의 상태. cleanable과 공유한다.
    private final State state;

    // cleanable 객체. 수거 대상이 되면 방을 청소한다.
    private final Cleaner.Cleanable cleanable;

    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);
        cleanable = cleaner.register(this, state);
    }

    @Override public void close() {
        cleanable.clean();
    }
}
```

`Teenage.class`

```java
package effectivejava.chapter2.item8;

import java.util.concurrent.TimeUnit;

// cleaner 안전망을 갖춘 자원을 제대로 활용하지 못하는 클라이언트 (45쪽)
public class Teenager {
    public static void main(String[] args) {
        new Room(99);
        System.out.println("Peace out");

        // 다음 줄의 주석을 해제한 후 동작을 다시 확인해보자.
        // 단, 가비지 컬렉러를 강제로 호출하는 이런 방식에 의존해서는 절대 안 된다!
//      System.gc();
    }
}
```
