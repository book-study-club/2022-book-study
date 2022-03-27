# 7. 오류 처리

- 오류 발생 가능성은 항상 존재
- 오류 처리 코드로 인해 프로그램 논리가 더러워져서는 안 됨

## 오류 처리 기법

### 오류 코드보다 예외를 사용하라

- 오류 코드를 던지는 경우, 항상 해당 함수 호출 시 오류 코드를 확인하는 로직을 사용해줘야 함
- 함수의 로직과 오류 처리 로직을 분리하라

### `try-catch-finally` 문부터 작성하라

- 이 덕분에 트랜잭션과 같은 형태로 로직을 구성할 수 있게 된다
- 강제로 예외를 일으키는 Test case를 먼저 작성하는 것도 좋은 방법

### Unchecked 예외를 사용하라

- 반드시 해당 예외를 처리해줘야 하는 Checked Exception을 사용하면...
  - 어떤 함수에서 새로운 Checked Exception을 throw 한다고 해보자
  - 그럼 이 함수를 사용하는 모든 곳에서 새로운 Checked Exception을 처리해줘야 하는 로직을 추가해줘야 함
- 다시말해, Check Exception은 함수를 사용할 때 내부 구현까지 모두 알아야 하기에 OCP(Open-Closed Principle) 위반

### 의미 있는 예외를 던져라

- 발생된 예외를 통해 원인과 위치를 찾을 수 있도록 하라
- 실패한 연산 이름과 실패 유형도 같이 언급

### 호출자(Caller)를 고려해 예외 클래스를 정해라

```java
// bad
Port port = new Port();

try {
  port.open();
} catch (DeviceResponseException e) {
  reportError(e);
} catch (UnlockedException e) {
  reportError(e);
} catch (DeviceRejectException e) {
  reportError(e);
} finally {
  // ...
}
```

```java
// good
Port port = new Port();

try {
  port.open();
} catch (PortFailure e) {
  reportError(e);
} finally {
  // ...
}
```

- 유사한 의미의 예외는 한 번에 묶어 throw 하자
  - 가령 위에서는 **서로 비슷한 의미를 가져 유사한 처리 로직을 갖는** `DeviceResponseException`, `UnlockedException`, `DeviceRejectException`이 있음
  - 이를 `PortFailure`라는 예외 하나만 해당 함수에서 던지도록 하면 Caller의 예외 처리 로직이 매우 간결해짐
  - 이를 위해 Callee 내부에서 예외를 Wrapping 하던가 할 수 있겠지

### 정상적인 흐름을 정의하라

```ts
// bad
let myCosts = 0;

try {
  // 한 사람이 모두 계산
  myCosts += expenses.getTotal();
} catch {
  // 각자 계산
  myCosts += expenses.getTotalByOne();
}
```

```ts
// good
let myCosts = 0;
myCosts += expenses.getCost(); // expenses 내부에서 알아서 처리
```

- 정말 필요한 경우에만 예외를 던지도록 하라
- 예외 처리 로직으로 인해 Caller의 코드가 더러워 질 수도 있다

### 예외 대신 `null`을 반환하지 마라

- 오류 발생을 Exception이 아닌 `null`을 반환하는 것으로 처리한다면?
  - 묵시적인(Implicit) 예외 처리 로직이 필요하기에 이는 좋지 않음
  - 예외 처리를 깜빡해버릴 수 있다는 것이다

### 함수의 인자로 `null`을 전달하지 마라

- 사용하는 함수(Callee) 내부에서 `null`을 처리하는 로직이 존재하지 않는다면 이는 바로 runtime error가 되어버림
- 물론 실제로 필요한 경우도 있긴 한데, 웬만하면 `null`에 의미를 부여하지 말라는 것

## Conclusion

- 깨끗한 코드는 가독성도 좋아야 하지만, 무엇보다 안정성도 높아야 함
