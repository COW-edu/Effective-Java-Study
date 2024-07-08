# 2장 객체 생성과 파괴

- 객체를 만들어야 할 때와 만들지 말아야 할 때를 구분하는 법
- 객체의 올바른 생성 방법과 불필요한 생성을 피하는 방법
- 파괴됨을 보장, 파괴 전 수행해야 할 정리 작업을 관리

## Item1. 생성자 대신 정적 팩터리 메서드를 고려하라

---

팩토리 메소드(Factory method)

- https://inpa.tistory.com/entry/GOF-💠-팩토리-메서드Factory-Method-패턴-제대로-배워보자

정적 팩토리 메소드(Static factory method)

- https://hstory0208.tistory.com/entry/OOP-정적-팩토리-메서드를-왜-사용하는가-어떤-상황에-사용하는게-좋을까-생성자와-차이

생성자

```java
public class Game {
    private String genre;
 
    public Game(String genre) {
        this.genre = genre;
    }
}
////////////////////////////////////////////
public static void main(String[] args) {
        Game game = new Game("FPS");
}
```

정적 팩토리 메소드

```java
public class Game {
    private String genre;
 
    private Game(String genre) {
    }
 
    public static Game from(String genre) {
        return new Game(genre);
    }
}
///////////////////////////////////////////
public static void main(String[] args) {
        Game game = Game.from("RPG");
}
```

정적 메서드

```java
public static Boolean valueOf (boolean b) {
		return b ? Bollean.TRUE : Boolean.FALSE;
}
```

### 정적 팩터리 메서드 장점

1. 이름을 가질 수 있다.
    - 네이밍을 통해 반환될 객체의 특성을 쉽게 묘사 가능
    - 생성자에 두 개 이상의 인자를 가졌을 때 순서를 알기 어려운 점을 해결
    
    ```java
    public class Game {
        private String genre;
        private String rating;
     
        private Game(String genre, String rating) {
            this.genre = genre;
            this.rating = rating;
        }
        
        public static Game adultAccessFrom(String genre) {
            return new Game(genre, "청소년이용불가");
        }
     
        public static Game fullAccessFrom(String genre) {
            return new Game(genre, "전체이용가");
        }
    }
    ```
    
2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.
    - Boolean.valueOf(boolean)은 객체를 생성하지 않기에 성능을 높힌다
    - 플라이웨이트 패턴(Flyweight pattern)
    - 싱글톤으로 활용 가능 → 단 모두 싱글톤은 아니다
3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
    
    ```java
    List<String> modifiableList = new ArrayList<>();
    modifiableList.add("a");
    modifiableList.add("b"); 
    
    List<String> unmodifiableList = Collections.unmodifiableList(modifiableList);
    
    ```

   ![Untitled](https://github.com/COW-edu/Effective-Java-Study/assets/68328998/13d13567-8f8b-42dd-b01d-ffd11cc08849)

    - List 인터페이스를 구현하고 있다.
    - 단순하게 사용할 수 있고, 유연성을 높힌다.
4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
    
    ```java
    public interface Car {}
    class SportsCar implements Car {}
    class SuvCar implements Car {}
     
    public static Car createCar(String type) {
        if (type.equals("sports")) {
            return new SportsCar();
        }
        if (type.equals("suv")) {
            return new SuvCar();
        } 
        throw new IllegalArgumentException("해당 타입의 자동차가 존재하지 않습니다 : " + type);
    }
    ```
    
5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
    
    ```java
    public class CarFactory {
     
        private CarFactory() {
        }
     
        public static Car createCar(String type) {
            if (type.equals("sports")) {
                return new SportsCar();
            }
            if (type.equals("suv")) {
                return new SuvCar();
            } 
            if (type.equals("electric")) {
                return new ElectricCar();
            }
            throw new IllegalArgumentException("해당 타입의 자동차가 존재하지 않습니다 : " + type);
        }
    }
    ```
    
    - client는 팩토리 메소드 호출만 하면 됨 → 어떤 클래스의 객체가 생성됐는지 알 필요 X
    - 유연한 확장성 / 내부 구현을 드러내지 않아 캡슐화 가능
    

### 정적 팩터리 메서드 단점

1. 상속하려면 public, protected 생성자 필요, 정적 팩토리 메소드만으로 하위 클래스 만들 수 없음
    - 상속 대신 합성으로 이를 해결
    
    ```java
    public class Car {
        private Car() {
        }
     
        public static Car createCar() {
            return new Car();
        }
    }
     
    public class SportsCar {
        private final Car car;
     
        public SportsCar(Car car) {
            this.car = car;
        }
    }
    ```
    
2. 정적 팩토리 메소드는 프로그래머가 찾기 어렵다
    - JAVADOC 문서를 찾아야한다
    
    ```java
    // from : 매개변수 하나를 받아 해당 타입의 인스턴스 반환
    Date d = Date.from(instance);
    
    // of : 매개 변수를 받아 적절한 타입의 인스턴스 반환
    Set<Rank> faceCards = EnumSet.of(JACK, QUEEN);
    
    // etc..
    ```
    

## Item2. 생성자에 매개변수가 많다면 빌더를 고려하라

---

점층적 생성자 패턴

```java
public class Person {
    
    private final String name;
    private final int age;
    private final String address;
    
    // 1개의 인자를 받는 생성자 (필수인자)
    public Person(String name) {
        this(name, 0);
    }
 
    // 2개의 인자를 받는 생성자
    public Person(String name, int age) {
        this(name, age, null);
    }
 
    // 3개의 인자를 모두 받는 생성자
    public Person(String name, int age, String address) {
        this.name = name;
        this.age = age;
        this.address = address;
    }
}
```

- 점층적 생성자 패턴은 확장하기 어렵다
- 매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵다
    - 매개변수의 순서 고려
    - 어떤 순서에 뭔지 모름

자바빈즈 패턴(JavaBeans pattern)

```java
Person person = new Person();
 
person.setName("developer");
person.setAge(29);
person.setAddress("Pangyo");

```

- 객체에 setter로 주입하는 것
- 일관성이 꺠지고, 불변으로 만들 수 없다.
    - setter 함수를 사용해 수정이 가능하기에
- 여러 개의 메소드 호출
- 일관성이 무너진 상태가 됨 → 객체 완성 이전까지

빌더 패턴(Builder pattern)

```java
public class Person {
 
    private final String name;
    private final int age;
    private final String address;
 
    public static class Builder {
        
        // 필수 인자
        private final String name;
 
        // 선택 인자 (default 값 셋팅)
        private int age = 0;
        private String address = "";
 
        public Builder(String name) {
            this.name = name;
        }
 
        public Builder age(int val) {
            age = val;
            return this;
        }
 
        public Builder address(String val) {
            address= val;
            return this;
        }
 
        public Person build() {
            return new Person(this);
        }        
    }
 
    public Person(Builder builder) {
        name = builder.name;
        age = builder.age;
        address = builder.address;
    }
}

```

```java
Person person = new Person().builder("developer")
    .age(29)
    .address("Pangyo")
    .build();

```

- 명명된 선택적 매개변수를 흉내낸 것
- 계층적으로 설계된 클래스와 함께 쓰기 좋다
- 생성자나 정적 팩토리가 처리해아 할 매개변수가 많다면 빌더 패턴을 선택하는 것이 낫다

✅ 불변(immutable)

- 어떠한 변경도 허용하지 않는다
- ex) String객체는 불변객체

## Item3. private 생성자나 열거 타입으로 싱글턴임을 보증하라

---

### 싱글톤(singleton)

> 인스턴스를 오직 하나만 생성할 수 있는 클래스
> 
- 싱글톤은 테스트하는데 어렵다
    - 타입을 인터페이스로 정의 및 구현하는 게 아니면
    - mock으로 대체할 수 없다.
    

### 싱글톤을 만드는 방법

1. public static final

```java
public static final Elvis INSTANCE = new Elvis();
```

- 장점
    - 싱글톤임이 명백히 들어남
    - 간결
    
1. 정적 팩터리 메서드

```java
public class Elvis{
    private static final Elvis INSTANCE = new Elvis();

    private Elvis() {
    }

    public static Elvis getInstance() {
        return INSTANCE;
    }
}
```

```java
private Object readResolve() {
	return INSTANCE;
}
```

- 직렬화된 인스턴스를 역직렬화 할 때 새로운 인스턴스가 생성된다. (동기화 작업 필요)
- 이것을 막기 위해 readResolve 메소드 추가

1. 열거형 방식

```java
public enum Elvis {
	INSTANCE;
	
	public void leaveTheBuilding (){
	}
}
```

- 코드가 간결해진다
- 직렬화 과정에서 문제가 없기에 good

## Item4. 인스턴스화를 막으려거든 prvate 생성자를 사용하라

---

- private 생성자를 추가하여 클래스의 인스턴스화를 막아라

why?

- 추상 클래스로 만드는 것은 인스턴스화를 막을 수 없음
- 하위 클래스를 만들어 인스턴스화 하면 됨

## Item5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

---

- 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식 → 의존 객체 주입 패

why?

- 사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글톤이 적합치 않다.

→ 의존성이 많은 큰 프로젝트에서 의존 객체 주입 프레임워크를 사용하여 이를 해결한다.

```java
public class SpellChecker {
    private final Lexicon dictionary;

    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public boolean isValid(String word) { 
        // implementation here 
    }

    public List<String> suggestions(String typo) { 
        // implementation here 
    }
}

```

## Item6. 불필요한 객체 생성을 피하라

---

- 하지 말아야 할 코드

```java
String s = new String("bikini");
```

→ 실행될 때 마다 인스턴스가 계속 생성된다.

- 문자열 지터럴을 사용하여 객체를 재사용하기

```java
String s = "hello";
```

- 정적 팩토리 메소드를 통해 불필요한 객체 생성 피하기

```java
public final class Boolean implements java.io.Serializable, Comparable<Boolean> {

    public static final Boolean TRUE = new Boolean(true);
    public static final Boolean FALSE = new Boolean(false);

    // ...
    public static boolean parseBoolean(String s) {
        return "true".equalsIgnoreCase(s);
    }

    // ...
    public static Boolean valueOf(String s) {
        return parseBoolean(s) ? TRUE : FALSE;
    }
}
```

- 값 비싼 객체는 캐싱을 통해 생성
    - 비싼 객체 : 인스턴스를 생성하는데 드는 비용이 크

```java
public class RomanNumber {
    private static final Pattern ROMAN = Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeral(String str) {
        return ROMAN.matcher(str).matches();
	}
}
```

- 오토 박싱을 사용할 때 주의하라
    - 오토박싱 : 기본 타입과 박싱 된 기본 타입을 자동으로 변환해주는 기술

```java
void autoBoxing_Test() {
    Long sum = 0L;
    for(long i = 0; i <= Integer.MAX_VALUE; i++) {
        sum += i;
    }
}

void noneBoxing_Test() {
    long sum = 0L;
    for(long i = 0; i < Integer.MAX_VALUE; i++) {
        sum += i;
    }
}
```

## Item7. 다 쓴 객체 참조를 해제하라

---

### 메모리 누수가 일어나는 곳은??

```java
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
}
```

- 스택이 커졌다 줄어들 때 스택에서 꺼낸 객체는 `GC`가 회수 하지 않는다
    - 여전히 스택이 다 쓴 참조를 들고있음
- 스택이 메모리를 직접 관리하기 때문이다.
- 따라서 자기 메모리를 직접 관리하는 클래스이면 메모리 누수에 주의 해야 함
- 캐시 역시 메모리 누수를 일으키는 주범
    
    ```java
    Map<Object, List> cache = new HashMap<>();
    cache.put(key1, value1);
    ```
    
    - 캐시 외부에서 key를 참조하는 동안만 엔트리가 살아있는 캐시가 필요한 상황이라면 `WeakHashMap`를 사용

POP 구현 → NULL처리

```java
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; // 다 쓴 참조 해제
    return result;
}
```

- 객체 참조를 null 처리하는 일은 예외적인 경우여야 한다

## Item8. finalizer와 cleaner 사용을 피하라

---

### finalizer란?

> finalize 메소드를 override하여 gc가 일어날 때 수행 되는 것
> 

### cleaner란?

> gc가 일어난 경우, 등록한 스레드에서 정의된 클린 작업을 수행 / 자원 반납용 안정망으로 사
> 

- finalizer는 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요하다.
- cleaner는 finalizer보다 덜 위험하지만, 여전히 예측할 수 없고, 느리고, 일반적으로 불필요하다.
- finalizer와 cleane로는 제때 실행되어야 하는 작업은 절대 할 수 없다
    - 시스템이 동시에 열 수 있는 파일 개수에 한계가 있기 때문에
- 상태를 영구적으로 수정하는 작업에서는 절대 finalize나 cleaner에 의존 해서는 안된다.
- 심각한 성능 문제도 동반한다
- finalizer 공격에 노출되어 심각한 보안 문제를 일으킬 수 있다

Adult

```java
public class Adult {
    public static void main(String[] args) {
        try (Room myRoom = new Room(7)) {
            System.out.println("안녕~");
        }
    }
}
```

Room

```java
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

Teenage

```java
public class Teenager {
    public static void main(String[] args) {
        new Room(99);
        System.out.println("아무렴");
    }
}
```

## Item9. try-finally보다 try-with-resources를 사용하라

---

### try-finally란?

> try 블록에서 일어난 일에 관계없이 항상 실행이 보장되어야 할 뒷정리용 코드
> 

try-finally방식

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
```

- try-finally방식을 사용했을 때 자원이 둘 이상이면 코드가 너무 지저분하다.
- 따라서 try-with-resources사용하는 것이 복수 자원을 사용하는데 짧고 매혹