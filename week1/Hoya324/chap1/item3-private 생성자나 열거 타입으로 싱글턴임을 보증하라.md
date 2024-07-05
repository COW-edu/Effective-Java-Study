# item3-private 생성자나 열거 타입으로 싱글턴임을 보증하라

## 핵심

### 열거 타입 방식의 싱글턴 - 바람직한 방법

- 원소가 하나인 열거 타입을 선언하자.

## 이유

### 싱글턴(sigleton)이란?

- 인스턴스를 오직 하나만 생성할 수 있는 클래스

### 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기 어려워진다.

### 싱글턴을 만드는 방식

1. private으로 감춰두고, 유일한 인스턴스에 접근할 수 있는 수단으로 public static 멤버를 하나 마련한다.
    
    ```java
    ...
    public static final Elvis INSTANCE = new Elvis();
    ...
    ```
    
    - 장점
        - 명백히 싱글톤임이 들어남
        - 간결함
2. 정적 팩터리 메서드를 public static 멤버로 제공
    
    ```java
    ...
    private static final Elvis INSTANCE = new Elvis();
    ...
    public static Elvis getInstance() { return INSTANCE; }
    ...
    ```
    
    - API를 바꾸지 않고도 싱글턴이 아니도록 변경 가능
    - 제네릭 싱글턴 팩터리로 만들 수 있음
    - 정적 팩터리의 메서드 참조를 공급자(supplier)로 사용할 수 있다는 점

이 두 가지 방식들 중 하나로 싱글턴 클래스를 직렬화하려면 단순히 Serializable을 구현한다고 선언하는 것만으로는 부족하다.

```java
private Object readResolve() {
	return INSTANCE;
}
```

이런식으로 진짜 Elvis를 반환하고 가짜 Elvis는 GC에게 맡긴다.

### 열거 타입 방식의 싱글턴 - 바람직한 방법

- 원소가 하나인 열거 타입을 선언하자.

```java
public enum Elvis {
	INSTANCE;
	
	public void leaveBuilding() { ... }
}
```

- 간단하게 직렬화도 가능
- 리플렉션 공격에도 제 2의 인스턴스를 생성하는 일을 완벽히 막아준다.
- 대부분의 상황에서 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법
