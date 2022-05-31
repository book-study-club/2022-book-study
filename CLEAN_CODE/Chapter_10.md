# Chapter 10 - 클래스

코드의 표현력과 그 코드로 이루어진 함수에 아무리 신경 쓸지라도 좀 더 차원 높은 단계까지 신경 쓰지 않으면 깨끗한 코드를 얻기는 어렵다. 이 장에서는 깨끗한 클래스를 다룬다.

## 클래스 체계

---

표준 자바 관례에 따르면, 가장 먼저 변수 목록이 나온다. 정적 공개, 정적 비공개 변수가 나오며, 이어서 비공개 인스턴스 변수가 나온다. 공개 변수가 필요한 경우는 거의 없다.
변수 목록 다음에는 공개 함수가 나오고, 비공개 함수는 자신을 호출하는 공개 함수 직후에 넣는다. 즉, **추상화 단계가 순차적으로 내려간다**.

### 캡슐화

변수와 유틸리티 함수는 가능한 공개하지 않는 편이 낫지만 반드시 숨겨야 한다는 법칙도 없다. 테스트 또한 중요하기 때문에, 테스트 코드가 함수를 호출하거나 변수를 사용해야 한다면 그 변수나 함수를 protected로 선언하거나 패키지 전체로 공개한다. 하지만 그 전에 비공개 상태를 유지할 방법을 강구한다. 캡슐화를 풀어주는 결정은 언제나 최후의 결정이다.

## 클래스는 작아야 한다!

---

클래스를 만들 때 제일 중요한 규칙은 크기이다. 클래스는 작아야 한다. 함수는 물리적인 행 수로 크기를 측정했지만, 클래스는 맡은 책임을 센다.

```java
// 메서드 수가 작음에도 불구하고 책임이 너무 많다.
public class SuperDashboard extends JFrame implements MetaDataUser {
	public Component getLastFocusedComponent()
	public void setLastFocused(Component lastFocused)
	public int getMajorVersionNumber()
	public int getMinorVersionNumber()
	public int getBuildNumber()
}
```

클래스 이름은 해당 클래스 책임을 기술해야 한다. 간결할 이름이 떠오르지 않는다면 클래스 크기가 너무 커서 그렇다. 클래스 이름이 모호하다면 클래스 책임이 너무 많아서다. 예를 들어, 클래스 이름에 Processor, Manager, Super 등과 같이 모호한 단어가 있다면 클래스에다 여러 책임을 떠안겼다는 증거다.

또한 클래스 설명은 if, and, or, but을 사용하지 않고 25단어 내외로 가능해야 한다.

### 단일 책임 원칙

단일 책임 원칙은 클래스나 모듈을 변경할 이유가 하나 뿐이어야 한다는 원칙이다. 책임, 즉 변경할 이유를 파악하려 애쓰다 보면 코드를 추상화하기도 쉬워진다.

```java
public class Version {
	public int getMajorVersionNumber()
	public int getMinorVersionNumber()
	public int getBuildNumber()
}
```

위의 SuperDashboard에서 버전 정보를 다루는 메서드 세 개를 따로 빼내 Version이라는 독자적인 클래스를 만들 수 있다.

SRP는 객체 지향 설계에서 중요하고 이해하고 지키기 수월한 개념이지만 클래스 설계자가 가장 무시하는 규칙 중 하나이다. 대다수는 ‘돌아가는 소프트웨어’에 초점을 맞추고 ‘깨끗하고 체계적인 소프트웨어’에는 관심을 두지 않는다.

큰 클래스 몇 개가 아니라 작은 클래스 여럿으로 이뤄진 시스템이 더 바랍직하다. 작은 클래스는 각자 맡은 책임이 하나며, 변경할 이유도 하나며, 다른 작은 클래스와 협력해 시스템에 필요한 동작을 수행한다.

### 응집도

클래스는 인스턴스 변수 수가 작아야 한다. 각 클래스 메서드는 클래스 인스턴스 변수를 하나 이상 사용해야 한다. 일반적으로 메서드가 변수를 더 많이 사용할수록 메서드와 클래스는 응집도가 높다.

모든 인스턴스 변수를 메서드마다 사용하는 클래스는 응집도가 가장 높지만, 이런 클래스는 가능하지도, 바람직하지도 않지만 가능한한 응집도가 높은 클래스를 지향해야 한다. 응집도가 높다는 말은 클래스가 속한 메서드와 변수가 의존하며 논리적인 단위로 묶인다는 의미기 때문이다.

```java
// Stack을 구현한 코드. 응집도가 아주 높다.

public class Stack {
	private int topOfStack = 0;
	List<Integer> elements = new LinkedList<Integer>();

	public int size() {
		return topOfStack;
	}

	public void push(int element) {
		topOfStack++;
		elements.add(element);
	}

	public int pop() throws PoppedWhenEmpty {
		if (topOfStack == 0)
			throw new PoppedWhenEmpty();
		int element = elements.get(--topOfStack);
		elements.remove(topOfStack);
		return element;
	}
}
```

`함수를 작게, 매개변수 목록을 짧게` 전략을 따르다 보면 때때로 몇몇 메소드만이 사용하는 인스턴스 변수가 아주 많아진다. 이는 새로운 클래스로 쪼개야 한다는 신호다. 응집도가 높아지도록 변수와 메서드를 적절히 분리해 새로운 클래스 두세 개로 쪼개준다.

**응집도를 유지하면 작은 클래스 여럿이 나온다.**

예를 들어, 변수가 아주 많은 큰 함수 하나를 작은 함수 하나로 빼내고 싶다. 그런데 빼내려는 코드가 큰 함수에 정의된 변수 넷을 사용한다면, 변수 네 개를 새 함수에 인수로 넘겨야 할까??
만약 네 변수를 클래스 인스턴스 변수로 승격한다면 새 함수는 인수가 필요없다. 그만큼 함수를 쪼개기 쉬워진다.

큰 함수를 작은 함수 여럿으로 쪼개다 보면 종종 작은 클래스 여럿으로 쪼갤 기회가 생긴다.

### 변경하기 쉬운 클래스

대다수 시스템은 지속적인 변경이 가해진다. 깨끗한 시스템은 클래스를 체계적으로 정리해 변경에 수반하는 위험을 낮춘다.

```java
// 새로운 SQL문을 지원할 때 손대야 하고 기존 SQL 문을 수정할 때도 손대야 한다.

public class Sql {
	public Sql(String table, Column[] columns)
	public String create()
	public String insert(Object[] fields)
	public String selectAll()
	public String findByKey(String keyColumn, String keyValue)
	public String select(Column column, String pattern)
	public String select(Criteria criteria)
	public String preparedInsert()
	private String columnList(Column[] columns)
	private String valuesList(Object[] fields, final Column[] columns) private String selectWithCriteria(String criteria)
	private String placeholderList(Column[] columns)
}
```

클래스 일부에서만 사용되는 비공개 메서드는 코드를 개선할 잠재적인 여지를 시사한다.

```java
// 공개 인터페이스를 각각 SQL 클래스에서 파생하는 클래스로 만들고, 비공개 메서드는 파생 클래스로 옮겼다.
// 공통으로 사용하는 비공개 메서드는 유틸리티 클래스에 넣었다.

abstract public class Sql {
	public Sql(String table, Column[] columns)
	abstract public String generate();
}

public class CreateSql extends Sql {
	public CreateSql(String table, Column[] columns)
	@Override public String generate()
}

public class SelectSql extends Sql {
	public SelectSql(String table, Column[] columns)
	@Override public String generate()
}

public class InsertSql extends Sql {
	public InsertSql(String table, Column[] columns, Object[] fields)
	@Override public String generate()
	private String valuesList(Object[] fields, final Column[] columns)
}

public class SelectWithCriteriaSql extends Sql {
	public SelectWithCriteriaSql(
	String table, Column[] columns, Criteria criteria)
	@Override public String generate()
}

public class SelectWithMatchSql extends Sql {
	public SelectWithMatchSql(String table, Column[] columns, Column column, String pattern)
	@Override public String generate()
}

public class FindByKeySql extends Sql public FindByKeySql(
	String table, Column[] columns, String keyColumn, String keyValue)
	@Override public String generate()
}

public class PreparedInsertSql extends Sql {
	public PreparedInsertSql(String table, Column[] columns)
	@Override public String generate() {
	private String placeholderList(Column[] columns)
}

public class Where {
	public Where(String criteria) public String generate()
}

public class ColumnList {
	public ColumnList(Column[] columns) public String generate()
}
```

이렇게 된다면 SQL문을 추가할 때 기존 클래스를 변경할 필요가 전혀 없다.

새 기능을 추가하거나 기존 기능을 변경할 때 건드릴 코드가 최소인 시스템 구조가 바람직하다.

### 변경으로부터 격리

객체 지향 프로그래밍에는 구체적인(concrete) 클래스와 추상(abstract) 클래스가 있는데 상세한 구현에 의존하는 클라이언트 클래스는 구현이 바뀌면 위험에 빠진다. 그래서 인터페이스와 추상 클래스를 사용해 구현이 미치는 영향을 격리한다.

테스트가 가능할 정도로 시스템의 결합도를 낮추면 유연성과 재사용성도 더욱 높아진다. 결합도가 낮다는 소리는 각 시스템 요소가 다른 요소로부터 그리고 변경으로부터 잘 격리되어 있다는 의미이다. 시스템 요소가 서로 잘 격리되어 있으면 각 요소를 이해하기도 더 쉬워진다.
