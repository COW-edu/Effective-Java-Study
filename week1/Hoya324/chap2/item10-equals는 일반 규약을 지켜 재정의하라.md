# item10-equals는 일반 규약을 지켜 재정의하라

## 핵심

꼭 필요한 경우가 아니라면 equals를 재정의하지 말자. 많은 경우에 Object의 equals가 우리가 원하는 비교를 정확하게 수행해준다.

재정의해야할 때는 IDE에게 맞기거나, 그 클래스의 핵심 필드를 모두 빠짐없이, 다섯 가지 규약을 확실히 지켜가며 비교해야 한다.

## 이유

### **equals 메서드**

- equals 메서드는 곳곳에 함정이 도사리고 있어서 자칫하면 끔찍한 결과를 초래한다.

### **재정의 하지 않는 것이 최선이다.**

**1. 각 인스턴스가 본질적으로 고유하다.**

- 값을 표현하는 게 아니라 동작하는 개체를 표현하는 클래스(Thread)

**Thread**

```java
@Override
public boolean equals(Object obj) {
    if (obj == this)
        return true;

    if (obj instanceof WeakClassKey) {
        Object referent = get();
        return (referent != null) && (referent == ((WeakClassKey) obj).get());
    } else {
        return false;
    }
}

```

**2. 인스턴스의 '논리적 동치성'을 검사할 일이 없다.**

- equals를 재정의하여 각 인스턴스가 같은 정규표현식을 나타내는지 검사할 수 있지만, 그럴 일이 없다면 굳이 재정의할 필요가 없다.(Pattern)

**Pattern**

```java
void patternTest() {
    final Pattern P1 = Pattern.compile("//.*");
    final Pattern P2 = Pattern.compile("//.*");

    System.out.println(P1.equals(P1)); // true
    System.out.println(P1.equals(P2)); // false
    System.out.println(P1.pattern()); // //.*
    System.out.println(P1.pattern().equals(P1.pattern())); // true
    System.out.println(P1.pattern().equals(P2.pattern())); // true
}
```

**3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.**

- Set의 구현체들은 모두 AbstractSet이 구현한 equals를 상속 받아서 쓴다. 예컨대 대부분의 Set 구현체는 AbstractSet이 구현한 equals를 상속받아서 쓴다.

**4. 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.**

```
@Override
public boolean equals(Object o){
    throw new AssertionError(); // 호출 금지
}
```

### **언제 equals를 재정의 할까?**

- 객체 식별성(object identity); 두 객체가 물리적으로 같은가) 이 아니라 논리적 동치성을 확인해야하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때다.

### 재정의할 때는 반드시 일반 규약을 따라야한다.

**equals의 일반 규약**

null이 아닌 모든 참조 값 x, y, z에 대해,

- 반사성:
    - x.equals(x) == true
- 대칭성:
    - x.equals(y) == true
    - > y.equals(x) == true
- 추이성:
    - x.equals(y) == true
    - > y.equals(z) == true
    - > x.equals(z) == true
- 일관성:
    - x.equals(y)를 반복해도 값이 변하지 않는다.
- null-아님:
    - x.equals(null) == false

> 존 던(John Donne)의 말처럼 세상에 홀로 존재하는 클래스는 없기에,
협력 관계에서 수 많은 클래스는 자신에게 전달된 객체가 equals 규약을 지킨다고 가정하고 동작한다.

**어렵지만 지켜야한다.**
> 

**1. 반사성**

- null이 아닌 모든 참조 값 x에 대해 x.equals(x)를 만족해야 한다.
- 자기 자신과 같아야 한다.

**2. 대칭성**

- null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)가 true이면, y.equals(x)가 true를 만족해야 한다.
- 서로에 대한 동치 여부가 같아야 한다.

```java
public final class CaseInsensitiveString {

    private final String str;

    public CaseInsensitiveString(String str) {
        this.str = Objects.requireNonNull(str);
    }

    @Override
    public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString) {
            return str.equalsIgnoreCase(((CaseInsensitiveString) o).str);
        }

        if (o instanceof String) { // 한 방향으로만 작동한다.
            return str.equalsIgnoreCase((String) o);
        }
        return false;
    }
}

void symmetryTest() {
    CaseInsensitiveString caseInsensitiveString = new CaseInsensitiveString("Test");
    String test = "test";
    System.out.println(caseInsensitiveString.equals(test)); // true
    System.out.println(test.equals(caseInsensitiveString)); // false
}
```

**3. 추이성**

- null이 아닌 모든 참조 값 x, y, z에 대해 x.equals(y)가 true이고, y.equals(z)가 true이면 x.equals(z)도 true이다.

```java
public class Point {

    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) {
            return false;
        }
        Point p = (Point) o;
        return this.x == p.x && this.y == p.y;
    }
}
```

```java
public class ColorPoint extends Point {

    private final Color color;

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) {
            return false;
        }

        // o가 일반 Point이면 색상을 무시햐고 x,y 정보만 비교한다.
        if (!(o instanceof ColorPoint)) {
            return o.equals(this);
        }

        // o가 ColorPoint이면 색상까지 비교한다.
        return super.equals(o) && this.color == ((ColorPoint) o).color;
    }
}
```

```java
void transitivityTest() {
    ColorPoint a = new ColorPoint(2, 3, Color.RED);
    Point b = new Point(2, 3);
    ColorPoint c = new ColorPoint(2, 3, Color.BLUE);

    System.out.println(a.equals(b)); // true
    System.out.println(b.equals(c)); // true
    System.out.println(a.equals(c)); // false
}
```

**3-1. 추이성(무한 재귀)**

```java
public class LoofPoint extends Point {

    private final Loof loof;

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) {
            return false;
        }

        // o가 일반 Point이면 색상을 무시햐고 x,y 정보만 비교한다.
        if (!(o instanceof LoofPoint)) {
            return o.equals(this);
        }

        // o가 ColorPoint이면 색상까지 비교한다.
        return super.equals(o) && this.loof == ((LoofPoint) o).loof;
    }
}

void infinityTest() {
    Point cp = new ColorPoint(2, 3, Color.RED);
    Point sp = new LoofPoint(2, 3, Loof.SWEET);

    System.out.println(cp.equals(sp));
}
```

**3-2. 추이성(리스코프 치환 원칙)**

만약 추이성을 지키기 위해서 Point의 equals를 각 클래스들을 getClass를 통해서 같은 구체 클래스일 경우에만 비교하도록 하면 어떨까?

```java
@Override
public boolean equals(Object o) {
    // getClass
    if (o == null || o.getClass() != this.getClass()) {
        return false;
    }

    Point p = (Point) o;
    return this.x == p.x && this.y = p.y;
}
```

이렇게 되면 동작은 하지만, 리스코프 치환원칙을 지키지 못하게 된다.

- 이는 모든 객체 지향 언어의 동치관계에서 나타나는 근본적인 문제이며,
객체 지향적 추상화의 이점을 포기하지 않는 한 불가능하다.

**3-3 추이성(상속 대신 컴포지션(아이템 18)을 사용해라)**

```java
public class ColorPoint2 {

    private Point point;
    private Color color;

    public ColorPoint2(int x, int y, Color color) {
        this.point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }

    public Point asPoint() {
        return this.point;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint)) {
            return false;
        }
        ColorPoint cp = (ColorPoint) o;
        return this.point.equals(cp) && this.color.equals(cp.color);
    }
}
```

**3-4 추이성(추상 클래스)**

- 추상 클레스의 하위 클래스에서라면 equals의 규약을 지키면서도 값을 추가할 수 있다.

**4. 일관성**

- null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.

```java
@Test
void consistencyTest() throws MalformedURLException {
    URL url1 = new URL("www.xxx.com");
    URL url2 = new URL("www.xxx.com");

    System.out.println(url1.equals(url2)); // 항상 같지 않다.
}
```

java.net.URL 클래스는 URL과 매핑된 host의 IP주소를 이용해 비교하기 때문에 같은 도메인 주소라도 나오는 IP정보가 달라질 수 있기 때문에 반복적으로 호출할 경우 결과가 달라질 수 있다.

**따라서 이런 문제를 피하려면 equals는 항시 메모리에 존재하는 객체만을 사용한 결정적 계산을 수행해야 한다.**

즉, 클래스가 불변이든 가변이든 equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안 된다.

**5. null-아님**

- null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false `o.equals(null) == true`인 경우는 상상하기 어렵지만, 실수로 `NullPointerException`을 던지는 코드 또한 허용하지 않는다.
- 위와 같은 검사가 아닌 instanceof를 사용한 묵시적 null 검사가 낫다

```java
@Override
public boolean equals(Object o) {
    if (!(o instanceof MyType)) {
        return false;
    }
}
```

**equals 좋은 재정의 방법**

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

    // float와 double을 제외한 기본 타입 필드는 == 를 사용한다.
    return this.x == p.x && this.y == p.y;

    // 필드가 참조 차입이라면 equals를 사용한다.
    return str1.equals(str2);

    // null 값을 정상 값으로 취급한다면 Objects.equals로 NullPointException을 예방하자.
    return Objects.equals(Object, Object);
}
```

## 관련 코드 및 예시 & Reference

- https://github.com/WegraLee/effective-java-3e-source-code/blob/master/src/effectivejava/chapter3/item10/inheritance/CounterPointTest.java
- [https://github.com/woowacourse-study/2022-effective-java/blob/main/03장/아이템_10/equals는_일반_규약을_지켜_재정의하라.md](https://github.com/woowacourse-study/2022-effective-java/blob/main/03%EC%9E%A5/%EC%95%84%EC%9D%B4%ED%85%9C_10/equals%EB%8A%94_%EC%9D%BC%EB%B0%98_%EA%B7%9C%EC%95%BD%EC%9D%84_%EC%A7%80%EC%BC%9C_%EC%9E%AC%EC%A0%95%EC%9D%98%ED%95%98%EB%9D%BC.md)
