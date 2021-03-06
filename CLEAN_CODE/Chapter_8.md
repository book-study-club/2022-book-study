# Chapter 8 - 경계

---

시스템에 들어가는 모든 소프트웨어를 직접 개발하는 경우는 드물다. 패키지를 사거나, 오픈 소스를 이용하기도 하고 때로는 사내 다른 팀이 제공하는 컴포넌트를 사용한다. 이런 외부 코드를 우리 코드에 깔끔하게 통합시켜야만 한다.

## 외부 코드 사용하기

---

인터페이스 제공자와 인터페이스 사용자 사이에는 특유의 긴장이 존재한다. 제공자는 적용성을 넓히려 애쓴다. 반면, 사용자는 자신의 요구에 집중하는 인터페이스를 바란다. 이런 긴장으로 인해 시스템 경계에서 문제가 생길 소지가 많다.

예를 들어, java.util.Map을 본다면,

- clear() void - Map
- containsKey(Object key) boolean – Map
- containsValue(Object value) boolean – Map
- entrySet() Set – Map
- equals(Object o) boolean – Map
- get(Object key) Object – Map
- getClass() Class<? extends Object> – Object
- hashCode() int – Map
- isEmpty() boolean – Map
- keySet() Set – Map

Sensor라는 객체를 담는 Map을 만든다면 다음과 같이 Map을 생성한다.

`Map sensors = new HashMap();`

Sensor 객체가 필요한 코드는 다음과 같이 Sensor 객체를 가져온다.

`Sensor s = (Sensor)sensors.get(sensorId);`

Map이 반환하는 Object를 올바른 유형으로 변환할 책임은 Map을 사용하는 클라이언트에 있다. 동작은 하지만 깨끗한 코드라 보기 어렵고, 의도도 분명히 드러나지 않는다. 대신 다음과 같이 제네릭을 사용하면 코드 가독성이 높아진다.

```java
Map<String, Sensor> sensors = new HashMap<Sensor>();

...

Sensor s = sensors.get(sensorId);
```

그렇지만 위 방법도 Map<String, Sensor>가 사용자에게 필요하지 않는 기능까지 제공하는 문제는 해결하지 못한다.

Map의 인터페이스가 바뀌거나 할 경우 또한 우리 코드의 많은 부분들이 바뀌어야 한다. 인터페이스가 바뀔 일이 별로 없을 것이라 생각할 지도 모르지만, 실제로 Java 5버전에서 generic이 추가되었을 때 Map의 인터페이스가 바뀐 사례가 있다.

결국 제일 좋은 방법은 래핑이다.

```java
public class Sensors {
	private Map sensors = new HashMap();

	public Sensor getById(String id) {
		return (Sensor) sensors.get(id);
	}
}
```

물론 Map 클래스를 사용할 때마다 캡슐화하라는 얘기가 아니다. Map을 여기저기 넘기지 말라는 말이다. Map과 같은 경계 인터페이스를 이용할 때는 이를 이용하는 클래스나 클래스 계열 밖으로 노출되지 않도록 주의한다.

## 경계 살피고 익히기

---

- 외부 코드를 사용할 때 우리가 사용할 코드를 테스트하는 편이 바람직하다.
- 우리쪽 코드를 작성해 외부 코드를 호출하는 대신 먼저 간단한 테스트 케이스를 작성해 외부 코드를 익히는 방법을 **학습 테스트**라 부른다.

## log4j 익히기

---

예를 들어 로깅 기능을 직접 구현하는 대신 아파치의 log4j 패키지를 사용한다고 가정한다.

```java
// 1. "hello" 출력 테스트
@Test
public void testLogCreate() {
	Logger logger = Logger.getLogger("MyLogger");
	logger.info("hello");
}

// 2. Appender가 필요하다는 오류 발생, ConsoleAppender 추가
@Test
public void testLogAddAppender() {
  Logger logger = Logger.getLogger("MyLogger");
  ConsoleAppender appender = new ConsoleAppender();
}

// 3. Appender에 출력 스트림이 없는 오류 발생
@Test
public void testLogAddAppender() {
  Logger logger = Logger.getLogger("MyLogger");
  logger.removeAllappenders();
  logger.addAppender(new ConsoleAppender(
          new PatternLayout("%p %t %m%n"),
          ConsoleAppender.SYSTEM_OUT));
  logger.info("hello");
}

// 4. 간단하게 테스트를 수정
public class LogTest {
  private Logger logger;

  @Before
  public void initialize() {
    logger = Logger.getLogger("logger");
    logger.removeAllAppenders();
    Logger.getRootLogger().removeAllAppenders();
  }

  @Test
  public void basicLogger() {
    BasicConfigurator.configure();
    logger.info("basicLogger");
  }

  @Test
  public void addAppenderWithStream() {
    logger.addAppender(new ConsoleAppender(
            new PatternLayout("%p %t %m%n"),
            ConsoleAppender.SYSTEM_OUT));
    logger.info("addAppenderWithStream");
  }

  @Test
  public void addAppenderWithoutStream() {
    logger.addAppender(new ConsoleAppender(
            new PatternLayout("%p %t %m%n")));
    logger.info("addAppenderWithoutStream");
  }
}
```

## 학습 테스트는 공짜 이상이다

---

학습 테스트는 드는 비용이 없다. 오히려 필요한 지식만 확보하는 손쉬운 방법이다. 학습 테스트는 이해도를 높여주는 정확한 실험이다.

학습 테스트는 패키지가 예상대로 도는지 검증한다. 패키지 새 버전이 나올 때마다 새로운 위험이 생긴다. 새 버전이 호환되지 않으면 학습 테스트가 이 사실을 곧ㄷ바로 밝혀낸다.

## 아직 존재하지 않는 코드를 사용하기

---

아직 개발되지 않은 모듈이 필요한데, 기능은 커녕 인터페이스조차 구현되지 않은 경우가 있을 수 있다. 구현을 진행하조가 인터페이스를 정의하는 것이 좋다.

인터페이스를 구현하면 인터페이스를 전적으로 통제한다는 장점이 생긴다. 또한 코드 가독성이 높아지고 코드 의도도 분명해진다.

## 깨끗한 경계

---

경계에서는 변경이 많이 벌어진다. 소프트웨어 설계가 우수하다면 변경하는데 많은 투자와 재작업이 필요하지 않다.

경계에 위치하는 코드는 깔끔히 분리한다. 또한 기대치를 정의하는 테스트 케이스도 작성한다. 통제가 불가능한 외부 패키지에 의존하는 대신 통제가 가능한 우리쪽 코드에 의존하는 편이 좋다.

외부 패키지를 호출하는 코드를 가능한 줄여 경계를 관리한다. 새로운 클래스로 경계를 감싸거나 어뎁터 패턴을 사용한다.
