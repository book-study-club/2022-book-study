# Chapter 4 - 주석

프로그래밍 언어를 치밀하게 사용해 의도를 표현할 능력이 있다면, 주석은 거의 필요하지 않을 것이다. 코드로 의도를 표현하지 못해서 실패를 만회하기 위해 주석을 사용한다.

이처럼 주석을 무시하는 이유는 프로그래머들이 주석을 유지하고 보수하기란 현실적으로 불가능하기 때문이다.

## 주석은 나쁜 코드를 보완하지 못한다.

---

코드에 주석을 추가하는 일반적인 이유는 코드 품질이 나쁘기 때문이다. 표현력이 풍부하고 깔끔하며 주석이 거의 없는 코드가, 복잡하고 어수선하며 주석이 많이 달린 코드보다 훨씬 좋다. 주석으로 설명하려 애쓰는 대신에 코드를 깔끔하게 작성하는데 시간을 써라.

## 코드로 의도를 표현하라.

---

```java
// before
// 직원에게 복지 혜택을 받을 자격이 있는지 검사한다.
if ((employee.flags & HOURLY_FLAG) && (employee.age > 65))

// after
if (employee.isEligibleForFullBenefits())
```

주석으로 달려는 설명을 함수로 만들어 표현해도 충분하다.

## 좋은 주석

---

### 법적인 주석

- 각 소스 파일 첫머리에 주석으로 들어가는 저작권 정보와 소유권 정보는 필요하고도 타당하다.

```jsx
// Copyright (C) 2003,2004,2005 by Object Mentor, Inc. All rights reversed.
// GNU General Public Licence 버전 2 이상을 따르는 조건으로 배포한다.
```

### 정보를 제공하는 주석

- 기본적으로 정보를 주석으로 제공하면 편리하다.

```java
// kk:mm:ss EEE, MMM dd, yyyy 형식이다.
Pattern timeMatcher = Pattern.compile(
	"\\d*:\\d*:\\d* \\w, \\w* \\d*, \\d*");
```

위에 제시한 주석은 코드에서 사용한 정규표현식이 시각과 날짜를 뜻한다고 설명한다.

### 의도를 설명하는 주석

- 주석은 구현을 이해하게 도와주는 섬을 넘어 결정에 깔린 의도까지 설명한다.

```java
// 스레드를 대량 생성하는 방법으로 어떻게든 경쟁 조건을 만들려 시도한다.
for (int i = 0; i < 25000; i++) {
	WidgetBuilderThread widgetBuilderThread =
		new WidgetBuilderThread(widgetBuilder, text, parent, failFlag);
	Thread thread = new Thread(widgetBuilderThread);
	thread.start();
}
```

### 의미를 명료하게 밝히는 주석

- 모호한 인수나 반환값은 그 의미를 읽기 좋게 표현하면 이해하기 쉬워진다.
- 인수나 반환값이 표준 라이브러리나 변경하지 못한느 코드에 속한다면 의미를 명료하게 밝히는 주석이 도움이 된다.
- 물론 그릇된 주석을 달아놓을 위험이 높기때문에, 달기 전에 더 나은 방법이 없는지 고민하고 정확히 달도록 조심해야 한다.

### 결과를 경고하는 주석

- 다른 프로그래머에게 결과를 경고할 목적으로 사용한다.

```java
// 여유 시간이 충분하지 않다면 실행하지 마십시오.
public void _testWithReallyBigFile() {
	writeLinesToFile(1000000);
	...
}

public static SimpleDateFormat makeStandardHttpDateFormat() {
	// SimpleDataFormat은 스레드에 안전하지 못하다.
	// 따라서 각 인스턴스를 독립적으로 생성해야 한다.
	SimpleDateFormat df = new SimpleDateFormat("EEE, dd MMM yyyy HH:mm:ss z");
	df.setTimeZone(TimeZone.getTimeZone("GMT"));
	return df;
}
```

### TODO 주석

- ‘앞으로 할 일’을 //TODO 주석으로 남겨두면 편하다.

```java
// TODO-MdM 현재 필요하지 않다.
// 체크아웃 모델을 도입하면 함수가 필요 없다.
protected VersionInfo makeVersion() throws Exception {
	return null;
}
```

- TODO 주석 전부를 찾아 보여주는 기능을 제공하는 IDE도 많기 때문에 주석을 잃어버릴 염려가 없다.

### 중요성을 강조하는 주석

- 대수롭지 않다고 여겨질 뭔가의 중요성을 강조하기 위해서도 주석을 사용한다.

```java
String listItemContent = match.group(3).trim();
// 여기서 trim은 정말 중요하다. trim 함수는 문자열에서 시작 공백을 제거한다.
// 문자열에 시작 공백이 있으면 다른 문자열로 인식되기 때문이다.
new ListItemWidget(this, listItemContent, this.level + 1);
return buildList(text.substring(match.end()));
```

### 공개 API에서 Javadocs

- 설명이 잘 된 공개 API는 참으로 만족스럽다.
- 공개 API를 구현한다면 반드시 훌륭한 Javadocs를 작성한다.

## 나쁜 주석

---

대다수의 주석이 나쁜 주석의 범주에 속한다. 일반적으로 대다수 주석은 허술한 코드를 지탱하거나, 엉성한 코드를 변명하거나, 미숙한 결정을 합리화하는 등 프로그래머가 주절거리는 독백에서 크게 벗어나지 못한다.

### 주절거리는 주석

- 이유없이 의무감으로 혹은 프로세스에서 하라고 하니까 마지못해 주석을 단다면 시간낭비이다.

```java
public void loadProperties() {
	try {
		String propertiesPath = propertiesLocation + "/" + PROPERTIES_FILE;
		FileInputStream propertiesStream = new FileInputStream(propertiesPath);
		loadedProperties.load(propertiesStream);
	} catch (IOException e) {
		// 속성 파일이 없다면 기본값을 모두 메모리로 읽어 들였단 의미다.
	}
}
```

catch 블록에 있는 주석의 의미는 저자에게는 의미가 있겠지만 다른 사람들에게는 의미가 전해지지 않는다. 답을 알아내기 위해서는 다른 코드를 뒤져보는 수박에 없다.

### 같은 이야기를 중복하는 주석

- 헤더에 달린 주석이 같은 코드 내용을 그대로 중복한다.

```java
// this.closed가 true일 때 반환되는 유틸리티 메서드다.
// 타임아웃에 도달하면 예외를 던진다.
public syncronized void waitForClose(final long timeoutMillies) throws Exception {
	if(!closed) {
		wait(timeoutMillis);
		if(!closed)
			throw new Exception("MockResponseSender could not be closed");
	}
}
```

### 오해할 의도가 있는 주석

- 의도는 좋으나 오해할 여지가 있는 주석이다.

### 의무적으로 다는 주석

- 모든 함수에 javadocs를 달거나 모든 변수에 주석을 달아야 한다는 규칙은 어리석다.
- 코드르 복잡하게 만들며, 거짓말을 퍼트리고 혼동과 무질서를 초래한다.

```java
/**
 *
 * @Param title CD 제목
 * @Param author CD 저자
 * @Param tracks CD 트랙 숫자
 * @Param durationInMinutes CD 길이(단위: 분)
 */
public void addCD(String title, String author, int tracks, int durationInMinutes) {
	CD cd = new CD();
	cd.title = title;
	cd.author = author;
	cd.tracks = tracks;
	cd.duration = durationInMinutes;
	cdList.add(cd);
}
```

### 이력을 기록하는 주석

- 때로 모듈을 편집할 때마다 모듈 첫머리에 주석을 추가한다.
- 현재는 소스 코드 관리 시스템이 있으니, 이제는 혼란만 가중할 뿐이다.

```java
* 변경 이력 (11-Oct-2011부터)
* -----------------------------
* 11-Oct-2001 : 클래스를 다시 정리하고 새로운 패키지로 옮겼다.
* 05-Nov-2001 : getDescription() 메서드 추가했으며 NotableDate class를 제거했다.
```

### 있으나 마나 한 주석

- 너무나 당연한 사실을 언급하며 새로운 정보를 제공하지 못한다.

```java
/**
 * 기본 생성자
 */
protected AnnualDateRule() {
}

/** 월 중 일자 */
private int dayOfMonth;
```

### 무서운 잡음

- 때로는 javadocs도 잡음이다.
- 문서를 제공해야한다는 잘못된 욕심으로 탄생한 잡음이다.

```java
/** The name. */
private String name;

/** The version */
private String version;

/** The licenceName */
private String licenceName;
```

### 함수나 변수로 표현할 수 있다면 주석을 달지 마라

```java
// 전역 목록 <smodule>에 속하는 모듈이 우리가 속한 하위 시스템에 의존하는가?
if (smodule.getDependSubsystems().contains(subSysMod.getSubSystem()))
```

이 코드에서 주석을 없애고 다시 표현할 수 있다.

```java
ArrayList moduleDependees = smodule.getDependSubSystems();
String ourSubSystem = subSysMod.getSubSystem();
if (moduleDependees.contains(ourSubSystem))
```

위와 같이 주석이 필요하지 않도록 코드를 개선하는 편이 좋다.

### 위치를 표시하는 주석

- 때로 소스 파일에서 특정 위치를 표시하기 위해 주석을 사용한다.

```java
// Actions ///////////////////////////////
```

이런 주석은 가독성만 낮추므로 제거하는게 좋다.

### 닫는 괄호에 다는 주석

- 중첩이 심하고 장황한 함수의 닫는 괄호에 특수한 주석을 달아 놓는다.
- 이러한 주석은 작고 캡슐화된 함수에는 잡음일 뿐이다.
- 대신에 함수를 줄이려 시도하는 것이 좋다.

### 공로를 돌리거나 저자를 표시하는 주석

- 이러한 정보는 소스 코드 관리 시스템에 정리하는 편이 좋다.

```java
/* 릭이 추가함 */
```

### 주석으로 처리한 코드

```java
InputStreamResponse response = new InputStreamResponse();
response.setBody(formatter.getResultStream(), formatter.getByteCount());
// InputStream resultsStream = formatter.getResultStream();
// StreamReader reader = new StreamReader(resultsStream);
// response.setContent(reader.read(formatter.getByteCount()));
```

주석으로 처리된 코드는 다른 사람들이 지우기를 주저한다. 그래서 점점 쓸모 없는 코드가 쌓여간다.

소스 코드 관리 시스템이 대신 코드를 기억해 줄 것이다.

### HTML 주석

- HTML 주석은 IDE에서조차 읽기 힘들다.

### 전역 정보

- 주석을 달아야 한다면 근처에 있는 코드만 기술한다.
- 전반적인 정보를 기술하지 마라.
- 코드가 변해도 주석이 변하라는 보장이 없기 때문이다.

```java
/**
 * 적합성 테스트가 동작하는 포트 : 기본값은 <b>8082</b>
 *
 * @param fitnessPort
 */
public void setFitnessPort(int fitnessPort)
{
	this.fitnessPort = fitnessPort;
}
```

### 너무 많은 정보

- 주석에다 흥미로운 역사나 관련 없는 정보를 장황하게 늘어놓지 마라.

### 모호한 관계

- 주석과 주석이 설명하는 코드는 둘 사이 관계가 명확해야 한다.

```java
/*
 * 모든 필섹을 담을 만큼 충분한 배열로 시작한다(여기에 필터 바이트를 더한다).
 * 그리고 헤더 정보를 위해 200바이트를 더한다.
 */
this.pngBytes = new byte[((this.width + 1) * this.height * 3) + 200];
```

여기서 필터 바이트는 무엇이며 +1, \*3과 관련이 있는지 알겠는가? 한 픽셀이 한 바이트가 맞는지 200을 더하는 이유는 무엇인지 모른다. 주석 자체가 다시 설명을 요구하게 된다.

### 함수 헤더

- 짧은 함수는 긴 설명이 필요없다.
- 짧고 한 가지만 수행하며 이름을 잘 붙인 함수가 주석으로 헤더를 추가한 함수보다 좋다.

### 비공개 코드에서 Javadocs

- 공개 API는 Javadocs가 유용하지만 공개하지 않을 코드라면 Javadocs는 쓸모 없다.
