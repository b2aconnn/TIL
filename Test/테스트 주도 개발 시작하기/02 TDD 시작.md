# TDD란?

## 의미

구현을 먼저 한 후에 테스트를 하는 것이 아니라 기능이 올바르게 동작하는 지 검증하는 **테스트 코드를 먼저 작성**



## 덧셈 기능 검증

### CalculatorTest

<img width="500" alt="스크린샷 2023-05-16 오후 11 12 43" src="https://github.com/b2aconnn/TIL/assets/101120568/9e0b2455-7722-4618-8273-e049ee7c841f">

- JUnit은 @Test 어노테이션을 메서드에 붙이면 테스트 코드로 인식
- assertEquals() 메소드는 인자로 값은 두 값을 동일한 지 비교. 동일하지 않을 시 AssertFailedError 발생

