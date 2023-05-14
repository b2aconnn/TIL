## TDD란?

구현을 먼저 한 후에 테스트를 하는 것이 아니라 기능이 올바르게 동작하는 지 검증하는 **테스트 코드를 먼저 작성**한다는 것을 의미



#### 덧셈 기능 검증

덧셈 기능을 검증하기 위한 테스트 코드 먼저 작성

```java
public class CalculatorTest {
    @Test
    void plus() {
        int result = Calculator.plus(1, 2);
        assertEquals(3, result);
        assertEquals(5, Calculator.plus(4, 1));
    }
}
```
