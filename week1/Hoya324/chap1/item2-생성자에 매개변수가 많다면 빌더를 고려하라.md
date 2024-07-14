# item2-생성자에 매개변수가 많다면 빌더를 고려하라

## 핵심

생성자나 정적 팩터리가 처리해야 할 매개 변수가 많다면, 빌더 패턴을 선택하는 것이 낫다.

매개변수 중 다수가 필수가 아니거나 같은 타입이라면 특히 더 그렇다. 빌더는 점층적 생성자보다 클라이언트 코드를 읽고 쓰기 훨씬 간결하고, 자바빈즈보다 훨씬 안전하다.

## 이유

### 1. 점층적 생성자 패턴도 쓸 수 있지만, 매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵다.

- 점층적 생성자 패턴은 생성자에 매개변수를 점차 늘려가며 필요한 생성자를 사용하는 것

### 2. 자바빈즈 패턴 - 일관성이 깨지고, 불변으로 만들 수 없다.

- 자바빈즈 패턴은 매개변수가 없는 생성자로 객체를 만들고 setter로 메서드들을 호출해 원하는 매개변수 값을 설정하는 방식
- 객체 하나를 만들려면 메서드를 여러 개 호출해야 하고, 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태에 놓임

### 3. 빌더 패턴 - 점층적 생성자 패턴의 안정성과 자바빈즈 패턴의 가독성을 겸비

- 필요한 객체를 직접 만드는 대신, 필수 매개변수만으로 생성자(혹은 정적 팩토리 메서드)를 호출해 빌더 객체를 얻는다.
- 만일 빌더 패턴에서 불변식을 보장하려면 빌더로부터 매개변수를 복사한 후 해당 객체 필드들도 검사해야한다.(item 50)

### 계층적으로 설계된 클래스와 잘 어울리는 빌더 패턴

`Pizza.class`

```java
package effectivejava.chapter2.item2.hierarchicalbuilder;
import java.util.*;

// 코드 2-4 계층적으로 설계된 클래스와 잘 어울리는 빌더 패턴 (19쪽)

// 참고: 여기서 사용한 '시뮬레이트한 셀프 타입(simulated self-type)' 관용구는
// 빌더뿐 아니라 임의의 유동적인 계층구조를 허용한다.

public abstract class Pizza {
    public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }

        abstract Pizza build();

        // 하위 클래스는 이 메서드를 재정의(overriding)하여
        // "this"를 반환하도록 해야 한다.
        protected abstract T self();
    }
    
    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone(); // 아이템 50 참조
    }
}
```

`NyPizza.class`

```java
package effectivejava.chapter2.item2.hierarchicalbuilder;

import java.util.Objects;

// 코드 2-5 뉴욕 피자 - 계층적 빌더를 활용한 하위 클래스 (20쪽)
public class NyPizza extends Pizza {
    public enum Size { SMALL, MEDIUM, LARGE }
    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        @Override public NyPizza build() {
            return new NyPizza(this);
        }

        @Override protected Builder self() { return this; }
    }

    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }

    @Override public String toString() {
        return toppings + "로 토핑한 뉴욕 피자";
    }
}
```

`Calzone.class`

```java
package effectivejava.chapter2.item2.hierarchicalbuilder;

// 코드 2-6 칼초네 피자 - 계층적 빌더를 활용한 하위 클래스 (20~21쪽)
public class Calzone extends Pizza {
    private final boolean sauceInside;

    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sauceInside = false; // 기본값

        public Builder sauceInside() {
            sauceInside = true;
            return this;
        }

        @Override public Calzone build() {
            return new Calzone(this);
        }

        @Override protected Builder self() { return this; }
    }

    private Calzone(Builder builder) {
        super(builder);
        sauceInside = builder.sauceInside;
    }

    @Override public String toString() {
        return String.format("%s로 토핑한 칼초네 피자 (소스는 %s에)",
                toppings, sauceInside ? "안" : "바깥");
    }
}
```

위처럼 하위 클래스에 빌더 패턴을 적용하면 가변인수 매겨변수를 여러 개 사용할 수 있다.

각각을 적절한 메서드로 나눠 선언하거나, 메서드를 여러 번 호출하도록 하고 각 호출 떄 넘겨진 매겨변수들을 하나의 필드로 모을 수도 있다.

## 고려 사항

### 빌더 생성 비용 - 성능에 민감한 상황에서는 문제가 될  수 있다.

### 빌더 패던 구현 코드가 장황하므로 4개 이상의 매개변수가 있을 때 그 값어치를 할 수도 있지만, 시간에 따라 매개변수는 증가하므로 초기부터 빌더패턴을 쓰는게 나을 수도 있다.

- Lombok을 사용하면 해결될 듯하다.

## 관련 코드 및 예시

https://github.com/WegraLee/effective-java-3e-source-code/blob/master/src/effectivejava/chapter2/item2/builder/NutritionFacts.java

```java
package effectivejava.chapter2.item2.builder;

// 코드 2-3 빌더 패턴 - 점층적 생성자 패턴과 자바빈즈 패턴의 장점만 취했다. (17~18쪽)
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // 필수 매개변수
        private final int servingSize;
        private final int servings;

        // 선택 매개변수 - 기본값으로 초기화한다.
        private int calories      = 0;
        private int fat           = 0;
        private int sodium        = 0;
        private int carbohydrate  = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings    = servings;
        }

        public Builder calories(int val)
        { calories = val;      return this; }
        public Builder fat(int val)
        { fat = val;           return this; }
        public Builder sodium(int val)
        { sodium = val;        return this; }
        public Builder carbohydrate(int val)
        { carbohydrate = val;  return this; }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize  = builder.servingSize;
        servings     = builder.servings;
        calories     = builder.calories;
        fat          = builder.fat;
        sodium       = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }

    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
                .calories(100).sodium(35).carbohydrate(27).build();
    }
}
```
