## Chapter 2 - 객체 생성과 파괴

### 1. 생성자 대신 정적 팩터리 메소드를 고려해라.

**장점**

1. 가시성이 좋다
    - 생성자로 객체를 생성하면 생성한 객체의 특성을 잘 고려하지 못함.
    - 정적 팩토리 메소드의 이름을 잘 지으면 반환될 객체의 특성을 쉽게 묘사할 수 있다.

2. 호출될 때 인스턴스의 생성을 선택할 수 있다.
    - 플라이웨이트 패턴
    - 인스턴스 통제 클래스(싱글톤, 인스턴스화 불가, 불변 값 클래스)

3. 반환 타입의 하위 타입 객체를 반환할 수 있다.
    - 인터페이스에 의존하게 만듦.
    - 과거 인터페이스에 정적 메소드를 선언할 수 없어, 동반클래스(companion class)를 만들어 그 안에 정의 하였다. (ex. java.util.Collections)

   → 자바 8부터 인터페이스가 정적 메소드를 가질 수 없다는 제한이 풀림


4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
    - ex. EnumSet 원소의 개수에 따라 반환하는 인스턴스가 다름.

5. 정적 팩토리 메소드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
    - 서비스 제공자 프레임 워크(service provider framework, ex. JDBC)
        - 서비스 인터페이스(service interface)
          : 구현체의 동작을 정의
          ex)Connection
          - 제공자 등록 API(provider registration API)
          : 제공자가 구현체를 등록
          ex) DriverManager.registerDriver
          - 서비스 접근 API(service access API)
          : 클라이언트가 서비스의 인스턴스를 얻을 때 사용
          ex) DriverManager.getConnection
          - 서비스 제공자 인터페이스
          :서비스 인터페이스의 인스턴스를 생성하는 팩터리 객체를 설명
          ex) Driver, Reflection API

**단점**

1. 상속하려면 public이나 protected 생성자가 필요하여 정적 팩토리 메소드만 제공하면 하위 클래스를 만들 수 없다.
2. 정적 팩토리 메소드는 프로그래머가 알기 쉽지 않음. 직접 만든 개발자만 기억하기 쉬움.

**유형**

- from
- of
- valueOf
- getInstance
- create / newInstance
- getType
  - 
- newType
- type

### 2. 생성자에 매개변수가 많다면 빌더를 고려해라

- 생성자로 매개변수를 받는 것은 가독성이 떨어지고 관리하기 어렵다.

**해결법**

1. 자바빈즈 패턴(JavaBeans Patter)
    - setter로 모든 데이터 입력하는 것
    - 호출해야하는 메소드가 너무 많음.
    - 일관성이 무너짐.
    - 클래스를 불변으로 만들 수 없음
2. 빌더 패턴(Builder Pattern)
    - 계층적으로 설계된 클래스에 유리하다.
    - 객체 생성에 유연하다는 장점을 가짐.
    - 빌더에 넘기는 매개변수에 따라 다른 객체를 생성함.
    - 매개변수 4개 이상부터 값어치를 함.
    - 단점은 빌더를 생성하는데 사용되는 비용

### 3. Private 생성자나 열거 타입으로 싱글턴임을 보장해라

- 싱글톤 : 인스턴스를 오직 하나만 생성할 수 있는 클래스
- readResolve메소드는 가짜 객체 생성을 예방해줌
  → 직렬화에서 역직렬화를 할 때 새로운 인스턴스가 생성됨.

```java
private Object readResolve() {
		//진짜 인스턴스를 반환하고, 가짜 인스턴스는 가비지 컬렉터에 맡김.
		return INSTANCE;
}
```

- 열거 타입의 싱글턴

```java
public enum Elvis {
	INSTANCE;
		
	public void leaveTheBuilding() {...}
}
```

→ 간결하고 좋다. 직렬화 상황이나 리플렉션 공격에도 제 2인스턴스 생성을 완벽히 막아줌.

### 4. 인스턴스화를 막으려면 private 메소드를 사용해라

### 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용해라(의존성 주입)

- 의존 객체 주입 패턴
- final 키워드를 제거하여 여러 의존성으로 교체하는 방법은 멀티스레드에 쓸 수 없다..

### 6. 불필요한 객체 생성을 피해라

- 생성자는 호출할 때마다 새로운 객체를 만들지만, 팩토리 메소드는 전혀 그렇지 않다.
- 불변 객체를 재사용하자.
- 기존 객체를 재사용해야 한다면 새로운 객체를 만들지 마라
- 새로운 객체를 만들어야 한다면 기존 객체를 재사용하지 마라

### 7. 다 쓴 객체 참조를 해제해라

- 자바가 가비지 컬렉터를 갖추고 있어 메모리 관리에 신경쓰지 않아도 되는거지, 가비지 컬렉터를 가지고 있지 않은 언어들은 메모리를 직접 관리해줘야 한다.
- 사용한 객체가 필요 없을 때 NULL 처리를 해라

### 8. Finalizer와 cleaner 사용을 피해라

- finalizer와 cleaner는 남은 리소스를 정리하기 위해 사용된다.
- finalizer는 예측이 불가능하고 위험할 수 있어 일반적으로 불필요.
- 현재 finalizer는 deprecated되어 있다.
- cleaner는 finalzer보다 덜 위험하지만, 똑같이 예측할 수 없고, 느리고 불필요하다.
- finalizer와 cleaner는 즉시 수행된다는 보장이 없어, 언제 실행되는 지 알 수 없고 원할 때 실행시키는 것은 힘들다.

**해결 방안**

- AutoClosable을 구현한다.
- try-with-Resource를 사용한다.

```java
public class ResourceManagementExample {

    public static void main(String[] args) {
        // 파일 읽기
        try (BufferedReader reader = new BufferedReader(new FileReader("input.txt"))) {
            String line;
            while ((line = reader.readLine()) != null) {
                System.out.println(line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

        // 파일 쓰기
        try (BufferedWriter writer = new BufferedWriter(new FileWriter("output.txt"))) {
            writer.write("Hello, world!");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### 9. try-finally보다는 try-with-resources를 사용해라

- 자바의 라이브러리를 사용한 뒤 close()를 사용해 자원을 닫아줘야 하는 경우가 많다.
- 하지만 클라이언트가 놓치기 쉬워, 보장하는 수단으로 try-finally가 사용되었었다.
- try-finally는 실용적이지 못할만큼 코드가 지저분해진다. 또한, try구문과 finally구문에서 모두 예외가 발생할 수 있는데, try 구문의 예외는 누락되고 finally구문에서의 예외처리만 되는 경우가 있다.
- try-with-resources는 예외 누락을 시키지 않고, 자동으로 자원을 회수해간다는 장점이 있다. 또한, 코드가 간결해진다는 장점이 있다.
- 회수해야 하는 자원을 다룰 때 꼭 try-with-resources를 사용하자.