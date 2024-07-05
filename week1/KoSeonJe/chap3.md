## Chapter 3 - 모든 객체의 공통 메소드

### 1. equals는 일반 규약을 지켜 재정의해라

- 객체 식별성이 아니라 논리적 동치성을 확인해야 한다.
- 값을 표현하는 클래스들은 equals를 재정의 해야한다.
- 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장한다면 equals를 재정의 하지 않아도 된다.
- equals 메소드 재정의 규약
    - 반사성
      : null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true이다.
    
    - 대칭성
      : null이 아닌 모든 참조 값 x,y에 대해, x.equals(y)가 true면 y.equals(x)도 true이다.
      
    - 추이성
      : null이 아닌 모든 참조 값 x,y,z에 대해, x.equals(y)가 true이고 y.equals(z)도 true이면 x.equals(z)도 true이다.
      
    - 일관성
      : null이 아닌 모든 참조 값 x,y에 대해 x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
      
    - null 아님
      : null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false이다.
      
    - 동치관계
      : 집합을 서로 같은 원소들로 이뤄진 부분집합으로 나누는 연산
      
    - == 과 equals()의 차이
      : ==는 주소값을 비교하는 것이고 equals()는 값 비교를 하는 것이다.


- equals() 재정의 시 비교 연산자
    - float와 double을 제외한 기본 연산자들은 == 연산자를 사용한다.
    - float와 double 필드는 각각 정적 메소드인 Float.compare(float,float)와 Double.compare(double, double)을 사용한다.
      → 그 이유는 부동 소수 값을 다뤄야 하기 때문이다.
    - 참조 타입은 equals()를 사용한다.
    - null값을 비교할 경우 NullPointerException 발생을 예방하기 위해 Objects.equals(Object, Object)를 사용한다.
    - 배열의 비교는 Arrays.equals()를 사용

- equals의 구현을 마치면 단위 테스트를 작성하여 돌려보는 것이 좋다.

- equals의 매개변수로 타입은 반드시 Object여야 한다.
- equals를 작성하고 테스트하는 일은 지루하고 테스트 코드도 뻔하기에 AutoValue 프레임워크를 사용하기도 한다. 애너테이션 하나만 추가하면 된다고 한다.

### 2. equals를 재정의하려거든 hashCode도 재정의 해라

- equals()가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.
- equals()를 재정의하여 논리적으로 동등하게 만들었지만, Obejct의 기본 hascode()는 둘이 전혀 다르다고 판단하여 다른 값을 반환한다.
- 이는 Hash를 사용한 자료구조에서 서로 다른 값이라고 인식한다.
- 두 객체가 다르다고 항상 다른 hashCode를 반환할 필요는 없지만(= 해시충돌), 다른 값을 반환해야 해시 테이블의 성능이 좋아진다.
- hashcode() 오버라이딩 방법

```java
@Override
public int hashCode() {
   return Objects.hash(field1,field2);
}
```

### 3. toString을 항상 재정의해라

- 기존 반환값은 “클래스_이름@16진수로_표시한_해시코드”
- toString을 항상 재정의 하는 것이 좋고, 재정의할 때 객체가 가진 주요 정보 모두를 반환하는 것이 좋다.
- 상위 클래스에서 이미 알맞게 재정의한 경우는 예외이다.
- 정적 유틸리티 클래스은 toString을 제공할 이유가 없다. 또한 Enum은 자바가 이미 완벽한 toString을 제공하니 따로 재정의 하지 않아도 된다.
- AutoValue 프레임워크가 toString도 생성해준다. 클래스의 정확한 의미는 전달해주지 못하지만, 해당 객체의 값들을 잘 알려준다.

### 4. clone 재정의는 주의해서 진행해라

- Cloneable
    - 복제해도 되는 클래스임을 명시해주는 인터페이스이다.
    - Cloneable을 구현하고 있는 클래스의 clone()은 Public으로 명시해주는 것이 좋다.
    - 문제점은 clone()이 선언된 곳은 Clonable이 아닌 Object이므로 Cloneable을 구현하는 것만으로 외부 객체에서 clone메소드를 호출할 수 없다.
    - 하지만 Cloneable 방식은 널리 쓰이고 있어 잘 알아두는 것이 좋다.

- clone() 일반 규약
    - x.clone() ≠ x
    - x.clone().getClass() == x.getClass()
    - x.clone().equals(x)

- super.clone()으로 clone()값을 반환해야 정상적이다. 생성자를 사용하는 것도 가능하지만, 하위클래스가 super.clone()을 사용한다면 잘못된 클래스의 객체가 만들어져 하위 클래스의 clone()이 제대로 동작하지 않는다.
  → final 클래스라면 위 이야기에 해당되지 않는다.

- 객체가 멤버 변수로 객체 배열을 가진 경우에 그대로 클론하면 원본 객체와 클론 객체의 멤버 변수가 같은 주소값을 가져 한쪽의 값을 변경하면 다른 쪽 값도 변경되는 일이 발생한다. 때문에 clone()을 구현할 때 배열의 clone()도 호출해줘야 한다.

    ```java
    @Override
    public Stack clone(){
    	try {
    			Stack result = (Stack} super.clone();
    			result.elements = elements.clone();
    			return result;
    	} catch (CloneNotSupportedException e) {
    			throw new AssertionError();
    	}
    }
    ```


- 때로는 위 방법만으로 충분하지 않고 deepCopy로 배열을 복사해야할 때가 있다.

→ Cloneable을 구현한 모든 클래스들은 clone()을 재정의 해줘야 한다. 가장 먼저 super.clone()을 호출한 후 필요한 필드를 전부 적절히 수정한다.

- 결론적으로 Cloneable/clone 방식보다 복사 생성자, 복사 팩토리(변환 생성자, 변환 팩토리)가 더 낫다.

```java
// 복사 생성자
public Yum(Yum yum) {...};

//복사 팩토리
public static Yum newInstance(Yum yum) {...};
```

### 5. Comparable을 구현할 지 고려하라

- Comparable 인터페이스
    - 구현한 클래스(자기자신)와 매개변수로 받은 클래스를 비교
    - compareTo()
        - 동치성 비교를 할 수 있지만, 주된 목적은 순서를 비교하는 것이다.
        - 두 객체 참조의 순서를 바꿔도 예상한 결과가 나와야 한다.
        - equals 규악과 같이 반사성, 대칭성, 추이성을 충족해야 한다.
- Comparator 인터페이스
    - 두 개의 객체를 매개변수로 받아 비교해준다.