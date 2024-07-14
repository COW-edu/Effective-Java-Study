# item1-생성자 대신 정적 팩터리 메서드를 고려하라

## 핵심

정적 팩터리 메서드와 public 생성자는 각자의 쓰임새가 있으니 상대적으로 장단점을 이해하고 사용할 것.

그러나 정적 팩터리를 사용하는 게 유리한 경우가 더 많으므로 무작정 public 생성자를 제공하던 습관은 고치자.

## 이유

### 1. 이름을 가질 수 있다.

- 반환될 객체의 특성을 쉽게 묘사

### 2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.

- 이로인해 **불변 클래스(Immutable class)**는 인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체 생성 피함
- 플라이웨이트 패턴(Flyweight pattern)과 비슷한 기법
- **인스턴스 통제(Instance-controlled)**반복되는 요청에 같은 객체를 반환하는 식으로 정적 팩터리 방식의 클래스는 언제 어느 인스턴스를 살아 있게 할지를 통제 가능

### 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.

- 반환 타입을 자유롭게 선택
- 응용하면 인터페이스 기반 프레임워크(item20)을 만드는 핵심기술 → 구현 클래스를 공개하지 않고도 그 객체 반환(API 작게 유지 가능)

```java
public interface ExampleInterface {
	ExampleResponse from(String arg);
}
```

- 이 interface만 알아도(구현 클래스를 보지 않더라도) 해당 클래스가 어떤 객체를 반환하는지 알 수 있다.

### 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

- 반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관없다.

### 5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

- JDBC같은 서비스 제공자 프레임워크(service provider framework)의 근간이 된다.
- **서비스 제공자 프레임워크(service provider framework) 핵심 컴포넌트**
    - 서비스 인터페이스(service interface)
        - 구현체의 동작을 정의
        - **JDBC의 Connecton**
    - 제공자 등록 API(provider rgistration API)
        - 제공자가 구현체를 등록할 때 사용
        - **JDBC의 DriverManager.registerDriver**
    - 서비스 접근 API(service access API)
        - 클라이언트가 서비스의 인스턴스를 얻을 때 사용
        - **JDBC의 DriverManager.getConnection**
    - 서비스 제공자 인터페이스(service provider interface)
        - 서비스 인터페이스의 인스턴스를 생성하는 팩터리 객체를 설명
        - 없으면 각 구현체를 인스턴스로 만들 때 리플렉션(item 65)를 사용
- **브리지 패턴(Bridge pattern)**처럼 서비스 제공자 프레임워크 패턴에는 여러 변형 존재

## 고려 사항

### 1. 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.

- 컬렉션 프레임워크의 유틸리티 구현 클래스들은 상속할 수 없다.

### 2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.

## 관련 코드 및 예시

- `from`: 매개변수 하나를 받아서 해당 타입의 인스턴스 반환
- of: 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드
    
    ```java
    Set<Rank> faceCards = EnumSet.of(JACK, Queen, King);
    ```
    
- `valueOf`: from과 of의 더 자세한 버전
- `instance` 혹은 getInstance: (매개변수를 받는다면) 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지 않는다.
- `create` 혹은 `newInstance`: instance 혹은 getInstance와 같지만, 매번 새로운 인스턴스를 생성해 반환함을 보장
- `get**Type**`: getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. “Type”은 팩터리 메서드가 반환할 객체의 타입이다.
    
    ```java
    BufferedReader br = Files.newBufferedReader(path);
    ```
    
- `type`: getType과 newType의 간결한 버전
    
    ```java
    List<Complaint> litancy = Collections.list(legacyLitancy);
    ```
