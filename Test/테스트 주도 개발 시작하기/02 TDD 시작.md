# TDD란?

## 의미

구현을 먼저 한 후에 테스트를 하는 것이 아니라 기능이 올바르게 동작하는 지 검증하는 **테스트 코드를 먼저 작성**



## 덧셈 기능 검증

### CalculatorTest

<img src="/Users/hjmac/Desktop/스크린샷 2023-05-16 오후 11.12.43.png" width="500" style="float: left"/>

- JUnit은 @Test 어노테이션을 메서드에 붙이면 테스트 코드로 인식
- assertEquals() 메소드는 인자로 값은 두 값을 동일한 지 비교. 동일하지 않을 시 AssertFailedError 발생

