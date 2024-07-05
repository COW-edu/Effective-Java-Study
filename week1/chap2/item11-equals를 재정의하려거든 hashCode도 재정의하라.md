# item11-equals를 재정의하려거든 hashCode도 재정의하라

## 핵심

equals를 재정의할 때는 hashCode도 반드시 재정의해야 한다. 그렇지 않으면 프로그램이 제대로 동작하지 않을 것이다.

재정의한 hashCode는 Object의 API 문서에 기술된 일반 규약을 따라야하며, 서로 다른 인스턴스라면 되도롣 해시코드도 서로 다르게 구현해야 한다.

## 이유

### equals를 클래스 모두에서 hashCode도 재정의해야한다.

- 그렇지 않으면 hashCode 일반 규약을 어기게 되어 해당 클래스의 인스턴스를 HashSet이나 HashMap 같은 컬렉션의 원소로 사용할 때 문제를 일으킬 것이다.

### hashCode 일반 규약

1. equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야한다.
    1. 단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관없다.
2. equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.
3. equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다.
    1. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

### hashCode 재정의를 잘못했을 때 크게 문제가 되는 조항은 두번째이다.

- 즉, 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.

### 성능을 높인답시고 해시코드를 계산할 때 핵심 필드를 생략해서는 안 된다.

### hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말자.

- 그래야 클라이언트가 이 값에 의지하지 않게 되고, 추후에 계산 방식을 바꿀 수도 있다.

## 관련 코드 및 예시

```java
package effectivejava.chapter3.item11;
import java.util.*;

// equals를 재정의하면 hashCode로 재정의해야 함을 보여준다. (70-71쪽)
public final class PhoneNumber {
    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = rangeCheck(areaCode, 999, "area code");
        this.prefix   = rangeCheck(prefix,   999, "prefix");
        this.lineNum  = rangeCheck(lineNum, 9999, "line num");
    }

    private static short rangeCheck(int val, int max, String arg) {
        if (val < 0 || val > max)
            throw new IllegalArgumentException(arg + ": " + val);
        return (short) val;
    }

    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof PhoneNumber))
            return false;
        PhoneNumber pn = (PhoneNumber)o;
        return pn.lineNum == lineNum && pn.prefix == prefix
                && pn.areaCode == areaCode;
    }

    // hashCode 없이는 제대로 동작하지 않는다. 다음 셋 중 하나를 활성화하자.

//    // 코드 11-2 전형적인 hashCode 메서드 (70쪽)
//    @Override public int hashCode() {
//        int result = Short.hashCode(areaCode);
//        result = 31 * result + Short.hashCode(prefix);
//        result = 31 * result + Short.hashCode(lineNum);
//        return result;
//    }

//    // 코드 11-3 한 줄짜리 hashCode 메서드 - 성능이 살짝 아쉽다. (71쪽)
//    @Override public int hashCode() {
//        return Objects.hash(lineNum, prefix, areaCode);
//    }

//    // 해시코드를 지연 초기화하는 hashCode 메서드 - 스레드 안정성까지 고려해야 한다. (71쪽)
//    private int hashCode; // 자동으로 0으로 초기화된다.
//
//    @Override public int hashCode() {
//        int result = hashCode;
//        if (result == 0) {
//            result = Short.hashCode(areaCode);
//            result = 31 * result + Short.hashCode(prefix);
//            result = 31 * result + Short.hashCode(lineNum);
//            hashCode = result;
//        }
//        return result;
//    }

    public static void main(String[] args) {
        Map<PhoneNumber, String> m = new HashMap<>();
        m.put(new PhoneNumber(707, 867, 5309), "제니");
        System.out.println(m.get(new PhoneNumber(707, 867, 5309)));
    }
}
```
