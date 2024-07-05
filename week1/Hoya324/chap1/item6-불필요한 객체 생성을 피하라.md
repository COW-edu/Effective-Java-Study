# item6-불필요한 객체 생성을 피하라

## 핵심

## 이유

### **1. 객채를 매번 생성하기보다는 재사용하는 편이 좋다**

- 단 하나의 인스턴스를 사용하면 객체의 재사용성이 보장된다.
- 만약, 무거운 객체라면 매번 생성할 때마다 많은 자원이 들어갈 것이고, 인스턴스를 자주 생성하게 되면 `GC`가 동작하게 될 확률이 높아지고 이는 애플리케이션 성능을 저하시키는 요인 중 하나이다.

**대표적인 예시 String** 

- [참조 링크](https://github.com/woowacourse-study/2022-effective-java/blob/main/02%EC%9E%A5/%EC%95%84%EC%9D%B4%ED%85%9C_06/%EB%B6%88%ED%95%84%EC%9A%94%ED%95%9C%20%EA%B0%9D%EC%B2%B4%20%EC%83%9D%EC%84%B1%EC%9D%84%20%ED%94%BC%ED%95%98%EB%9D%BC.md)

만약 `String`이 가변(mutable)이라면 같은 참조를 가지는 객체의 값을 변경할 수 있게 된다. 이는 같은 참조 & 다른 값이라는 상황을 만들 수 있기 때문에 재사용이 불가능하다.

따라서 객체 공유를 통한 재사용이 목적인 `String Pool`을 사용할 수 없다.

또한, 가변 상태의 객체는 같은 문자열 리터럴을 가지더라도 객체를 매번 생성해야 한다. 즉, new String(”str”)과 같이 쓸데없는 인스턴스를 만드는 경우 애플리케이션의 성능에 안 좋은 영향을 준다.

따라서, String을 불변으로 만들어 위와 같은 단점들을 제거하고 성능을 개선할 수 있게 된다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/fee67429-4153-4f2a-9f5f-751b1a73dc03/7ae9bf2f-43da-441b-9512-39056e918976/Untitled.png)

### 정적 팩터리 메서드를 활용하면 불필요한 객체 생성을 피할 수 있다.

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

- `Boolean.valueOf(String)`정적 팩터리 메서드를 사용하여 미리 생성된 `TRUE`, `FALSE`를 반환하는 것을 확인할 수 있다.

### 생성 비용이 아주 비싼 객체를 반복해서 사용하지 말아라

```java
package effectivejava.chapter2.item6;
import java.util.regex.Pattern;

// 값비싼 객체를 재사용해 성능을 개선한다. (32쪽)
public class RomanNumerals {

    // 코드 6-2 값비싼 객체를 재사용해 성능을 개선한다.
    private static final Pattern ROMAN = Pattern.compile(
            "^(?=.)M*(C[MD]|D?C{0,3})"
                    + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeralFast(String s) {
        return ROMAN.matcher(s).matches();
    }

    public static void main(String[] args) {
        int numSets = Integer.parseInt(args[0]);
        int numReps = Integer.parseInt(args[1]);
        boolean b = false;

        for (int i = 0; i < numSets; i++) {
            long start = System.nanoTime();
            for (int j = 0; j < numReps; j++) {
                // 성능 차이를 확인하려면 xxxSlow 메서드를 xxxFast 메서드로 바꿔 실행해보자.
                b ^= isRomanNumeralSlow("MCMLXXVI");
            }
            long end = System.nanoTime();
            System.out.println(((end - start) / (1_000. * numReps)) + " μs.");
        }

        // VM이 최적화하지 못하게 막는 코드
        if (!b)
            System.out.println();
    }
    
    
    // 코드 6-1 성능을 훨씬 더 끌어올릴 수 있다! -> 호출마다 객체 생성
    static boolean isRomanNumeralSlow(String s) {
        return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
                + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    }
}

```

### 오토박싱을 사용할 때 주의하라

```java
import org.junit.Test;

public class AutoBoxingTest {
	
	@Test
	public void autoBoxing_Test() {
		Long sum = 0L;
		for (long i = 0; i <= Integer.MAX_VALUE; i++) {
			sum += i;
		}
	}

	@Test
	public void noneBoxing_Test() {
		long sum = 0L;
		for (long i = 0; i < Integer.MAX_VALUE; i++) {
			sum += i;
		}
	}
}

```

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FKMxKs%2FbtsIn6jgxcM%2FefY44wXv2KF2ZxJn4YIcU0%2Fimg.webp)

Long → long (auto boxing)하는 비용에 의해 불필요하게 성능이 나빠질 수 있다.

즉, 박싱된 기본 타입보다는 기본 타입을 사용하고 의도치 않은 오토박싱이 숨어들지 않도록 주의하자.

## 고려 사항

### 객체 생성은 비싸니 피하라는 것이 아니다

- 프로그램의 명확성, 간결성, 기능을 위해서 객체를 추가로 생성하는 것은 좋은 일이다.

### 나만의 객체 풀은 삼가하자

- 예로 String pool 이 heap 영역에 이미 있는데 굳이 새로 인스턴스를 생성해서 사용하지 않을 이유가 없다.

### item 50은 “새로운 객체를 만들어야한다면 기존 객체를 재사용하지 마라”를 다룬다.

- 방어적 복사에 대해 알 수 있다.
