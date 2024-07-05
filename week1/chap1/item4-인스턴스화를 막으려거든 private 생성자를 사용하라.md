# item4-인스턴스화를 막으려거든 private 생성자를 사용하라

## 핵심

의도치 않게 클래스가 인스턴스화 되는 것을 막기 위해서는 prviate 생성자를 추가해라

## 이유

추상 클래스로 만든다고 하더라도 하위 클래스를 만들어 인스턴스화 하면 같은 문제가 생긴다.

## 관련 코드 및 예시

```java
// 기본 생성자가 만들어지는 것을 막기 위함
public class ExampleClass {
	private ExampleClass() {
		throw new AssertionError();
	}
}
```

- 주석을 다는 것도 좋은 방법이다.
