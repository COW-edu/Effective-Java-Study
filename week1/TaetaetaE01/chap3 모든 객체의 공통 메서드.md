# 3장 모든 객체의 공통 메서드

- final이 아닌 Object 메서드들을 언제 어떻게 재정의 해야하는지를 다룸

## Item10. equals는 일반 규약을 지켜 재정의하라

---

1. 각 인스턴스가 본질적으로 고유하다
    - 값을 표현하는 게 아니라 동작하는 캐채 표현(Tread)
2. 인스턴스의 논리적 동치성을 검사할 일이 없다
    - equals를 재정의해 인스턴스가 같은 정규 표현식을 나타내는지 검사 할 수 있지만 애초에 필요 없다고 판단할 수도 있음
    - 필요 없다 판단하면 equals만으로 해결 가능
3. 상위 클래스의 재정의한 equals가 하위 클래스에도 딱 들어 맞는다
    - set 구현체는 abstractSet 구현한 equals를 상속 받아 사용
    - List 구현체는 AbstractList로부터, Map은 AbstractMap으로 부터 상속
4. 클래스가 private이거나 package-private이면 equals 메서드를 호출할 일이 없다
    
    ```java
    @Override
    public boolean equals(Object o){
        throw new AssertionError(); // 호출 금지
    }
    ```
    

### equals메소드 규약

언제 제정의를 해야할까?

- 논리적인 동치성을 확인하고 싶을 때 재정의
- 예외
    - 값 클래스라고 해도 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장할 때

메소드 일반 규약

- 반사성 : null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true이다.
- 대칭성 : null이 아닌 모든 참조 값 x,y에 대해, x.equals(y)가 true면 y.equals(x)도 true이다.
- 추이성 : null이 아닌 모든 참조 값 x,y,z에 대해, x.equals(y)가 true이고 y.equals(z)도 true이면 x.equals(z)도 true이다.
- 일관성 : null이 아닌 모든 참조 값 x,y에 대해 x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.

### equals 좋은 재정의 방법

```java
@Override
public boolean equals(final Object o) {
    // 1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
    if (this == o) {
        return true;
    }

    // 2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
    if (!(o instanceof Point)) {
        return false;
    }

    // 3. 입력을 올바른 타입으로 형변환 한다.
    final Piece piece = (Piece) o;

    // 4. 입력 개체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사한다.
    
    // float와 double을 제외한 기본 타입 필드는 ==를 사용한다.
    return this.x == p.x && this.y == p.y;
    
    // 필드가 참조 차입이라면 equals를 사용한다.
    return str1.equals(str2);
    
    // null 값을 정상 값으로 취급한다면 Objects.equals로 NullPointException을 예방하자.
    return Objects.equals(Object, Object);
}
```

## Item11. equals를 재정의하려거든 hashCode도 재정의하라

### equals를 재정의한 클래스 모두에서 hashCode도 재정의 해야 함

- hashCode의 일반 규약을 어겨 HashMap, HashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으킴
- 규약
    - equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다.
        - 단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관없다.
    - equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.
    - equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다.
        - 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

### hashCode 재정의를 잘못했을 때 크게 문제가 되는 조항은 두번째이다

- 즉 논리적으로 같은 객체는 같은 해시 코드를 반환해야한다

### 성능을 높인답시고 해시코드를 계산할 때 핵심 필드를 생략해서는 안된다.

- 속도는 빨라지겠지만 품질이 나빠진다

### hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말자.

- 그래야 클라이언트가 이 값에 의지하지 않게 되고, 추후에 계산 방식을 바꿀 수도 있다.

## Item12. toString을 항상 재정의하라

- toString을 잘 구현한 클래스는 사용하기에 훨씬 즐겁고, 그 클래스를 사용한 시스템은 디버깅하기 쉽다.
- 실전에서 toString은 그 객체가 가진 주요 정보 모두를 반환하는게 좋다
- 포멧을 명싷다ㅡㄴ 아니든 여러분의 의도는 명확히 밝혀라

## Item13. clone 재정의는 주의해서 진행하라

### cloneable

- 복제해도 되는 클래스임을 명시하는 인터페이스
    - cloneable 인터페이스 : `clone`의 동작 방식을 결정하는 곳
        - 복사한 객체를 반환한다
        - 그렇지 못하면 `CloneNotSupportedException`을 던진다
- clone 메소드가 선언된 곳이 Object protected

### clone 메소드의 일반 규약

**1. `x.clone() != x`**

- 이 식은 참이다.

**2.`x.clone().getClass() == x.getClass()`**

- 이 식도 참이지만 반드시 만족해야 하는 것은 아니다.

**3. `x.clone().equals(x)`**

- 이 식도 일반적으로 참이지만, 필수는 아니다.

### clone 메소드의 재정의

1.  접근 제한자는 `public`으로 한다.
2. 반환 타입은 클래스 자신으로 변경한다.
3. 이 메서드는 가장 먼저 `super.clone` 을 호출한다.
4. 필요한 필드를 전부 적절히 수정한다.

가변 객체를 참조하지 않는 경우

```java
@Override
public PhoneNumber clone() {
    try {
        return (PhoneNumber) super.clone();
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();	
    }
}
```

가변 객체를 참조하는 경우

```java
public class Stack implements Cloneable {
	
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
    	if(size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; 
        return null;
    }
    
    private void ensureCapacity() {
    	if(elements.length == size)
            elements = Arrays.copyOf(elements, 2*size+1);
    }
}
```

## Item14. Comparable을 구현할지 고려하라