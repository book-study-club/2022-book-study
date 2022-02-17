# Chapter 2 - 의미 있는 이름

## 목차

---

- 의미 있는 이름의 중요성
- 코드에 그릇된 단서를 남기면 안된다
- 의미있게 구분하라
- 발음하기 쉬운 이름
- 검색하기 쉬운 이름
- 인코딩을 피해라
- 자신의 기억력을 자랑하지 마라
- 기발한 이름은 피해라
- 한 개념에 한 단어를 사용하라
- 말장난을 하지 마라
- 해법 영역에서 가져온 이름을 사용하라
- 문제 영역에서 가져온 이름을 사용하라
- 의미있는 맥락을 추가하라
- 불필요한 맥락을 없애라

## 의미 있는 이름의 중요성

---

- 의도가 분명한 이름을 지어야 한다
- 변수나 함수, 클래스의 이름은 존재 이유, 수행 기능, 사용 방법들을 명시해야 한다
- 좋은 이름 짓는데 고민하는 시간보다 이로 인해 절약되는 시간이 더 많다
- 의도가 분명한 이름의 중요성 예시 1)
  ```java
  public List<int[]> getThem() {
  	List<int[]> list1 = new ArrayList<int[]>();
  	for (int[] x : theList)
  		if (x[0] == 4)
  			list1.add(x);
  	return list1;
  }
  ```
  복잡한 문장 없이 단순한 코드지만 문제는 단순성이 아니라 함축성이다. 코드의 맥락이 명시적으로 들어나있지 않아서 theList에 무엇이 들었는지, 0번째 인덱스 값이 뭔지, 4가 왜 중요한지 뭐 아무것도 없다. 하지만 여기서 변수 이름을 통해 정보 제공이 충분히 가능했다. 이를 조금 고쳐본다면
  ```java
  public List<int[]> getFlaggedCells() {
  	List<int[]> flaggedCells = new ArrayList<int[]>();
  	for (int[] cell : gameBoard)
  		if (cell[STATUS_VALUE) == FLAGGED)
  			flaggedCells.add(cell);
  	return flaggedCells;
  }
  ```
  이러면 이제 게임판에서 깃발 상태를 찾는.. 지뢰찾기란걸 알수있다! 이걸 더 간단하게 한다면 FLAGGED 상수 대신해서 isFlagged 메소드를 만들어서 표현도 가능하다.

## 코드에 그릇된 단서를 남기면 안된다

---

- 잘못된 이름 사용으로 햇갈리거나 잘못된 정보를 제공해서는 안된다
- 널리 사용되는 약어를 변수 이름으로 사용한다면 안된다
- 또한 List가 아닌데도 묶었다는 의미로 변수명에 list가 들어가는 등.. 햇갈릴 수 있는 이름을 사용하면 안된다
- 비슷한 이름을 사용하는데 주의가 필요하다

## 의미있게 구분하라

---

- 연속된 숫자를 붙이거나 불용어를 사용하는 방식은 적절하지 않다
  - 이런 방식은 의미도 불명확하며 정보도 제공 못하는 변수가 된다
  - Info나 Data와 같은 불용어 사용 금지
- 접두어, 단수 or 복수형 등 의미가 불명확하면 사용하지 않는 것이 좋다

## 발음하기 쉬운 이름

---

- 읽기 힘들면 커뮤니케이션이 힘들다

## 검색하기 쉬운 이름

---

- 상수나 이름이 알파벳 하나인 변수는 검색하기 쉽지 않고 눈에 잘 띄지도 않는다
- 이름의 길이는 범위 크기에 비례하는 것이 좋다
  - 예를 들어 짧은 함수의 반복문 속 변수는 짧은 i나 j를 사용하는 것이 오히려 이해가 쉽다
  - 그러나 스코프가 커지면 의미를 잃게 되고 상황에 따라 다르게 정하는것이 좋다

## 인코딩을 피해라

---

- 이름에 불필요한 정보들을 넣지 말라는 의미
- 변수 앞에 변수 타입을 써놓는다거나, 변수에 접두어를 붙인다거나 등등
- 인터페이스와 구현 클래스로 나눌 때, ShapeFactoryImpl 또는 CShapeFactory 등이 좋다
  - 이 책에서는 IShapeFactory와 ShapeFactory같은 방식은 피하자고 한다

## 자신의 기억력을 자랑하지 마라

---

- 코드를 읽으면서 변수를 자기가 이해하기 쉬운 이름으로 변환해서 기억해야 한다면 좋은 이름이 아니다
- 명료함이 최고며, 모두가 이해하기 쉬운 이름으로 짓는 것이 중요하다
- 좋은 예 : postPayment, deletePage, save
- 나쁜 예 : Manager, Processor, Data, Info
- 접근자, 변경자, 조건자는 get, set, is로 시작하자
- 생성자를 중복 정의할 때는 메소드 인수를 설명하는 이름을 가지고 있는 정적 펙토리 메소드가 좋다

## 기발한 이름은 피해라

---

- 유머가 섞인 이름같이 기발한 이름의 경우 아는 사람들끼리만 기억하게 된다
- 명료하고 의도가 분명한 이름이 좋다

## 한 개념에 한 단어를 사용하자

---

- 추상적인 개념 하나에 단어 하나를 선택해 이를 사용해야 한다
- fetch, retrieve, get과 같이 제각각 사용하면 햇갈릴 수 있다
- controller, manager, driver를 섞어 쓰면 구분하기 쉽지 않을 것이다

## 말장난을 하지마라

---

- 한 단어를 두 가지 목적으로 사용하지 말고, 다른 개념에 같은 단어를 사용하면 안된다
- 이해하기 쉬운 코드가 목적이기 때문에 의도를 정확히 밝혀야 한다

## 해법 영역에 가져온 이름을 사용하라

---

- 읽는 사람도 프로그래머이기 때문에 프로그래머에게 익숙한 기술 개념에서 이름을 가져와 사용하는 것도 좋다

## 문제 영역에서 가져온 이름을 사용하라

---

- 적절한 프로그래밍 용어가 없다면 문제 영역에서 가져와도 괜찮다

## 의미있는 맥락을 추가해라

---

- 이름은 클래스, 함수 등에 의해서 맥락이 부여된다
- 그래도 불분명하다면 접두어를 붙여 맥락을 강조한다
- 예시 1)

```java
// 나쁜 예시
private void printGuessStatistics(char candidate, int count) {
	String number;
	String verb;
	String pluralModifier;
	if (count == 0) {
		number = "no";
		verb = "are";
		pluralModifier = "s";
	}  else if (count == 1) {
		number = "1";
		verb = "is";
		pluralModifier = "";
	}  else {
		number = Integer.toString(count);
		verb = "are";
		pluralModifier = "s";
	}
	String guessMessage = String.format("There %s %s %s%s", verb, number, candidate, pluralModifier );

	print(guessMessage);
}

// 좋은 예시
public class GuessStatisticsMessage {
	private String number;
	private String verb;
	private String pluralModifier;

	public String make(char candidate, int count) {
		createPluralDependentMessageParts(count);
		return String.format("There %s %s %s%s", verb, number, candidate, pluralModifier );
	}

	private void createPluralDependentMessageParts(int count) {
		if (count == 0) {
			thereAreNoLetters();
		} else if (count == 1) {
			thereIsOneLetter();
		} else {
			thereAreManyLetters(count);
		}
	}

	private void thereAreManyLetters(int count) {
		number = Integer.toString(count);
		verb = "are";
		pluralModifier = "s";
	}

	private void thereIsOneLetter() {
		number = "1";
		verb = "is";
		pluralModifier = "";
	}

	private void thereAreNoLetters() {
		number = "no";
		verb = "are";
		pluralModifier = "s";
	}
}
```

## 불필요한 맥락을 없애라

---

- 의미가 분명한 경우에는 짧은 이름이 긴 이름보다 좋다
- 불필요한 맥락을 넣어서 억지로 길게 만들거나, 쓸모없는 접두어의 사용은 피하자
