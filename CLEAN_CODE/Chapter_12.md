# Chapter 12 - 창발성

창발성이란 하위 계층(구성 요소)에는 없는 특성이나 행동이 상위 계층(전체 구조)에서 자발적으로 돌연히 출현하는 현상이다.

## 창발적 설계로 깔끔한 코드를 작성하자

켄트 벡이 제시한 단순한 설계 규칙 네 가지가 소프트웨어 설계 품질을 크게 높여준다고 믿는다.

- 모든 테스트를 실행한다.
- 중복을 없앤다.
- 프로그래머 의도를 표현한다.
- 클래스와 메소드 수를 최소로 줄인다.

### 모든 테스트를 실행하라

설계는 의도한 대로 돌아가는 시스템을 내놓아야 한다.

결합도가 높으면 테스트 케이스를 작성하기 어렵다. 그러므로 테스트 케이스를 많이 작성할수록 개발자는 DIP와 같은 원칙을 적용하고 의존성 주입, 인터페이스, 추상화 등과 같은 도구를 사용해 결합도를 낮춘다.

`테스트 케이스를 만들고 계속 돌려라`라는 간단하고 단순한 규칙을 따르면 시스템은 낮은 결합도와 높은 응집력이라는, 객체 지향 방법론이 지향하는 목표를 저절로 따르게된다.

### 리펙토링

리팩토링 단계에서는 소프트웨어 설계 품질을 높이는 기법이라면 무엇이든 적용해도 괜찮다. 응집도를 높이고, 결합도를 낮추고, 관심사를 분리하고, 시스템 관심사를 모듈로 나누고, 함수와 클래스 크기를 줄이고, 더 나은 이름을 선택하는 등 다양한 기법을 동원한다.

### 중복을 없애라

우수한 설계에서 중복은 커다란 적이다. 예를들어, 집합 클래스에 다음 메서드가 있다고 가정하자.

```java
int size() {
	boolean isEmpty() {}
}

boolean isEmpty() {
  return 0 == size();
}
```

isEmpty 메서드에서 size 메서드를 이용하면 코드를 중복해 구현할 필요가 없어진다.

```java
// as-is
public void scaleToOneDimension(float desiredDimension, float imageDimension) {
  if (Math.abs(desiredDimension - imageDimension) < errorThreshold)
    return;
  float scalingFactor = desiredDimension / imageDimension;
  scalingFactor = (float)(Math.floor(scalingFactor * 100) * 0.01f);

  RenderedOpnewImage = ImageUtilities.getScaledImage(image, scalingFactor, scalingFactor);
  image.dispose();
  System.gc();
  image = newImage;
}

public synchronized void rotate(int degrees) {
  RenderedOpnewImage = ImageUtilities.getRotatedImage(image, degrees);
  image.dispose();
  System.gc();
  image = newImage;
}

// to-be
public void scaleToOneDimension(float desiredDimension, float imageDimension) {
  if (Math.abs(desiredDimension - imageDimension) < errorThreshold)
    return;
  float scalingFactor = desiredDimension / imageDimension;
  scalingFactor = (float) Math.floor(scalingFactor * 10) * 0.01f);
  replaceImage(ImageUtilities.getScaledImage(image, scalingFactor, scalingFactor));
}

public synchronized void rotate(int degrees) {
  replaceImage(ImageUtilities.getRotatedImage(image, degrees));
}

private void replaceImage(RenderedOpnewImage) {
  image.dispose();
  System.gc();
  image = newImage;
}
```

아주 적은 양이지만 공통적인 코드를 새 메서드로 뽑고 보니 클래스가 SRP를 위반한다. 그러므로 새로 만든 replaceImage 매서드를 다른 클래스로 옮겨도 좋겠다. 그러면 새 메서드의 가시성이 좋아진다. 이런 ‘소규모 재사용’은 시스템 복잡도를 극적으로 줄여준다.

TEMPLATE METHOD 패턴은 고차원 중복을 제거할 목적으로 사용되는 패턴이다.

```java
// as-is
public class VacationPolicy {
  public void accrueUSDDivisionVacation() {
    // 지금까지 근무한 시간을 바탕으로 휴가 일수를 계산하는 코드
    // ...
    // 휴가 일수가 미국 최소 법정 일수를 만족하는지 확인하는 코드
    // ...
    // 휴가 일수를 급여 대장에 적용하는 코드
    // ...
  }

  public void accrueEUDivisionVacation() {
    // 지금까지 근무한 시간을 바탕으로 휴가 일수를 계산하는 코드
    // ...
    // 휴가 일수가 유럽연합 최소 법정 일수를 만족하는지 확인하는 코드
    // ...
    // 휴가 일수를 급여 대장에 적용하는 코드
    // ...
  }
}

// to-be
abstract public class VacationPolicy {
  public void accrueVacation() {
    caculateBseVacationHours();
    alterForLegalMinimums();
    applyToPayroll();
  }

  private void calculateBaseVacationHours() { /* ... */ };
  abstract protected void alterForLegalMinimums();
  private void applyToPayroll() { /* ... */ };
}

public class USVacationPolicy extends VacationPolicy {
  @Override protected void alterForLegalMinimums() {
    // 미국 최소 법정 일수를 사용한다.
  }
}

public class EUVacationPolicy extends VacationPolicy {
  @Override protected void alterForLegalMinimums() {
    // 유럽연합 최소 법정 일수를 사용한다.
  }
}
```

### 표현하라

소프트웨어 프로젝트 비용 중 대다수는 장기적인 유지보수에 들어간다. 코드를 변경하면서 버그의 싹을 심지 않으려면 유지보수 개발자가 시스템을 제대로 이해해야 한다. 하지만 시스템이 점차 복잡해지면서 유지보수 개발자가 시스템을 이해하느라 보내는 시간은 점점 늘어나고 동시에 코드를 오해할 가능성도 점점 커진다. 그러므로 코드는 개발자의 의도를 분명히 표현해야 한다. 개발자가 코드를 명백하게 짤수록 다른 사람이 그 코드를 이해하기 쉬워진다. 그래야 결함이 줄어들고 유지보수 비용이 적게 든다.

우선, 좋은 이름을 선택한다. 이름과 기능이 완전히 딴판인 클래스나 함수로 개발자를 놀라게 해서는 안 된다.

둘째, 함수와 클래스 크기를 가능한 줄인다. 작은 클래스와 작은 함수는 이름을 짓기도 쉽고, 구현하기도 쉽고, 이해하기도 쉽다.

셋째, 표준 명칭을 사용한다. 예를 들어, 디자인 패턴은 의사소통과 표현력 강화가 주요 목적이다. 클래스가 COMMAND나 VISITOR와 같은 표준 패턴을 사용해 구현된다면 클래스 이름에 패턴 이름을 넣어준다. 그러면 다른 개발자가 클래스 설계 의도를 이해하기 쉬워진다.

넷째, 단위 테스트 케이스를 꼼꼼히 작성한다. 테스트 케이스는 소위 '예제로 보여주는 문서'다. 다시 말해, 잘 만든 테스트 케이스를 읽어보면 클래스 기능이 한눈에 들어온다.

하지만 표현력을 높이는 가장 중요한 방법은 노력이다. 흔히 코드만 돌린 후 다음 문제로 직행하는 사례가 너무도 흔하다. 나중에 읽을 사람을 고려해 조금이라도 읽기 쉽게 만드려는 충분한 고민은 거의 찾기 어렵다. 하지만 나중에 코드를 읽을 사람은 바로 자신일 가능성이 높다는 사실을 명심하자.

### 클래스와 메서드 수를 최소로 줄여라

중복을 제거하고, 의도를 표현하고, SRP를 준수한다는 기본적인 개념도 극단으로 치달으면 득보다 실이 많아진다. 그래서 이 규칙은 함수와 클래스 수를 가능한 한 줄이라고 제안한다.

때로는 무의미하고 독단적인 정책 탓에 클래스 수와 메서드 수가 늘어나기도 한다. 클래스마다 무조건 인터페이스를 생성하라고 요구하는 구현 표준이 좋은 예다. 자료 클래스와 동작 클래스는 무조건 분리해야 한다고 주장하는 개발자도 좋은 예다. 가능한 독단적인 견해는 멀리하고 실용적인 방식을 택해야 한다.

목표는 함수와 클래스 크기를 작게 유지하면서 동시에 시스템 크기도 작게 유지하는 데 있다. 하지만 이 규칙은 간단한 설계 규칙 네 개 중 우선순위가 가장 낮다. 다시 말해, 클래스와 함수 수를 줄이는 작업도 중요하지만, 테스트 케이스를 만들고 중복을 제거하고 의도를 표현하는 작업이 더 중요하다는 뜻이다.
