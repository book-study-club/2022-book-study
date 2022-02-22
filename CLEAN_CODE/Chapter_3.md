# 3. 함수

아래 코드는 함수의 **나쁜** 예.

```ts
// code 1

const testableHtml = (
  pageData: PageData,
  includeSuiteSetup: boolean,
): string => {
  const wikiPage: WikiPage = pageData.getWikiPage();
  const buffer: string[] = [];

  if (pageData.hasAttribute('Test')) {
    if (includeSuiteSetup) {
      const suiteSetup: WikiPage = PageCrawlerImpl.getInheritedPage(
        SuiteResponder.SUITE_SETUP_NAME,
        wikiPage,
      );

      if (suiteSetup !== null) {
        const pagePath: WikiPagePath = suiteSetup.getPageCrawler().getFullPath(suiteSetup);
        const pagePathName: string = PathParser.render(pagePath);
        buffer.push(...[
          '!include -setup .',
          pagePathName,
          '\n',
        ]);
      }
    }

    const setup: WikiPage = PageCrawlerImpl.getInheritedPage('SetUp', wikiPage);

    if (setup !== null) {
      const setupPath: WikiPagePath = wikiPage.getPageCrawler().getFullPath9setup);
      const setupPathName: string = PathParser.render(setupPath);
      buffer.push(...[
        '!include -setup .',
        setupPathName,
        '\n',
      ]);
    }
  }

  buffer.push(pageData.getContent());

  if (pageData.hasAttribute('Test')) {
    const teardown: WikiPage = PageCrawlerImpl.getInheritedPage('TearDown', wikiPage);

    if (teardown !== null) {
      const teardownPath: WikiPagePath = wikiPage.getPageCrawler().getFullPath(teardown);
      const teardownPathName: string = PathParser.render(teardownPath);
      buffer.push(...[
        '!include -teardown .',
        teardownPathName,
        '\n',
      ]);
    }

    if (includeSuiteSetup) {
      const suiteTeardown: WikiPage = PageCrawlerImpl.getInheritedPage(
        SuiteResponder.SUITE_TEARDOWN_NAME,
        wikiPage,
      );

      if (suiteTeardown !== null) {
        const pagePath: WikiPagePath = suiteTeardown.getPageCrawler().getFullPath(suiteTeardown);
        const pagePathName: string = PathParser.render(pagePath);
        buffer.push(...[
          '!include -teardown .',
          pagePathName,
          '\n',
        ]);
      }
    }
  }

  pageData.setContent(buffer.join(''));
  return pageData.getHtml();
};
```

문제점은 이렇다.

- 다양한 추상화 수준(Level)
- 긴 함수의 코드
- 명확하지 않은 플래그(`includeSuiteSetup`, ...)
- 알 수 없는 문자열(`!include`)
- 예측되지 않는 함수를 사용(`getInheritedPage`, `getPageCrawler` ...)

이 코드의 의미를 대략적으로 유추할 수는 있겠으나, 상기된 문제점들로 인해 실행하기 전 까지는 그것이 실제 로직과 동일한지 알기가 매우 어렵다.

위 코드는 아래와 같이도 표현이 가능하다.

```ts
// code 2

const renderPageWithSetupsAndTeardowns = (
  pageData: PageData,
  isSuite: boolean,
): string => {
  const isTestPage = pageData.hasAttribute('Test');

  if (isTestPage) {
    const testPage: WikiPage = pageData.getWikiPage();
    const newPageContent: string[] = [];

    includeSetupPage(testPage, newPageContent, isSuite);
    newPageContent.push(pageData.getContent());
    includeTeardownPages(testPage, newPageContent, isSuite);

    pageData.setContent(newPageContent.join(''));
  }

  return pageData.getHtml();
}
```

이 코드는 *code 1* 보다 매우 간결하며, 또 그 의미를 유추하기도 더 쉽다. 또한 이름이 명확한 함수로 각 로직이 나뉘어져 있기에, 실제 동작 또한 유추된 의미와 동일할 것임을 쉽게 알 수 있다.

이게 어떻게 가능했을까?

## 함수 작성 가이드라인

- 작게 만들어라!
- 한 가지만 해라!
- 함수 당 추상화 수준은 하나로!
- 서술적인 이름을 사용하라!
- 함수 인수를 적게 사용하라!
- Side-Effects를 일으키지 말아라!
- 명령과 조회를 분리하라!
- 오류 코드보다 예외를 사용하라!
- 반복하지 마라!

### 작게 만들어라!

- 중첩된 if/else 또는 while문을 사용할 정도로 함수가 길어져서는 안된다

### 한 가지만 해라!

> 함수는 한 가지 일만을 해야 하며, 그것을 잘 해야 한다.

- 하나의 함수는 추상화 수준이 한 단계로 통일되어 있어야 한다.
- 어떤 로직이 하나의 함수로 추출될 수 있다면 그 로직을 포함하는 함수는 추상화 수준이 여러개인 것이다.

### 함수 당 추상화 수준은 하나로!

> *code 1* 은 여러 추상화 수준이 뒤섞인 코드라고 할 수 있고, *coded 2* 는 하나의 추상화 수준만이 기술된 함수라고 할 수 있다.

- 한 함수 내 추상화 수준을 섞으면 가독성이 매우 떨어지게 된다.
- 함수는 위에서 아래로 자연스럽게 이야기처럼 읽혀야 좋다.

다만 `switch` 또는 여러 `if/else`가 이어지는 구문은 이를 지키기 어렵다.

```ts
const calculatePay = (employee: Employee) => {
  switch (employee.type) {
    case COMMISSIONED:
      return calculateCommissionedPay(employee);
    case HOURLY:
      return calculateHourlyPay(employee);
    case SALARIED:
      return calculateSalariedPay(employee);
    default:
      throw new Error(employee.type);
  }
};
```

위 코드는 SRP(Single Responsibility Principle)과 OCP(Open Closed Principle) 둘 다 위반한다. 웬만하면 지양하도록 한다.

### 서술적인 이름을 사용하라!

> *code 1* 의 `testableHtml`이라는 이름의 함수를 *code 2* 에서 `renderPageWithSetupsAndTeardowns` 이름으로 바꾸어주었다.

- 코드를 읽으면서 짐작했던 기능을 각 루틴이 그대로 수행하도록 한다.
- 이름이 길어도, 정하느라 시간을 들여도 괜찮다.
- 모듈 내에서 함수 이름은 같은 문구, 명사, 동사를 사용해 도메인을 통일하도록 한다.

### 함수 인수(Argument)를 적게 사용하라!

- 인수가 많을아질수록 함수의 가독성이 떨어지며 테스트 또한 어려워진다.
- 인수를 하나만 전달하는 단항 형식 함수 예
  - 인수에 질문을 던지는 경우 (`fileExists(path: string) => boolean`)
  - 인수를 변환해 결과를 반환하는 경우 (`fileOpen(path: string) => InputStream`)
  - 가능한 인수를 수정하는 것을 피하고, 위 두 경우가 아니라면 단항 함수는 사용하지 않는다.
- 인수를 두 개 전달하는 이항 함수 (또는 삼항 함수)
  - Coord와 같이 자연적인 경우가 아니라면 인자가 두 개 이상인 함수는 순서를 기억해야 하기에 좋지 않다.
  - 인자가 두 개 이상인 함수는 가능한 단항 함수로 만들자.
- 플래그 인수는 함수 내에서 두 가지 이상을 처리하겠다는 의미를 가지기에, 이를 사용하기보다 각 로직을 함수로 나눈다.
- 인수를 의미 있는 데이터 타입으로 묶을 수 있는 경우 그렇게 한다.
  - 가령 `makeCircle(x: number, y: number, radius: number)`는 `makeCircle(center: Pointer, radius: number)`로 묶어줄 수 있다.
- 인자를 함수 이름에 명시할 수도 있다.
  - 단항 함수: 동사/명사 쌍으로 표기 (write + field = `writeField(name)`)
  - 이항 함수: 함수 이름에 키워드를 추가 (`assertEqual(expected, actual)` -> `assertExpectedEqualsActual(expected, actual)`)

### Side-Effects를 일으키지 말아라!

- 함수 내에서 부수 효과(Side-Effects)가 발생된다면, 이를 함수 이름에 명시하는 것이 좋다.
  - e.g. 세션을 초기화 하는 코드가 있는 패스워드 체크용 함수라면 -> `checkPasswordAndInitializeSession`
- 함수의 인수를 함부로 변경하지 않도록 한다.

### 명령과 조회를 분리하라!

```ts
// bad: 하나의 함수에서 명령과 조회를 동시에 실행

if (set('username', 'user-1')) {
  // ...
}
```

```ts
// good: 명령과 조회를 각기 다른 함수가 실행

if (isAttributeExists('username')) {
  set('username', 'user-1');
  // ...
}
```

- 함수는 무언가를 수행하거나 무언가를 답하거나 둘 중 하나만 한다.
  - e.g. 객체 상태를 변경하거나, 객체 정보를 반환하거나

### 오류 코드보다 예외를 사용하라!

```ts
// bad: 하나의 함수에서 명령과 조회를 실행

if (deletePage(page) === ERROR_OK) {
  // ...
}
```

```ts
// not-bad: 정상 로직과 오류 처리 로직이 뒤섞인 함수

try {
  deletePage(page);
  // success logic...
} catch (error) {
  log.error(error);
  // error logic...
}
```

```ts
// good: 오류 처리 로직을 함수로 분리

const tryDeletePage = (page: Page) => {
  try {
    deletePageAndDoSomething(page);
  } catch (error) {
    catchPageError(error);
  }
}
```

- 오류 코드를 반환하는 방식은 명령/조회 규칙을 위반한다.
  - 오류 코드 대신 예외 처리(`try-catch`)를 하면 코드가 깔끔해진다.
- 예외 처리는 정상 동작과 오류 처리 동작이 뒤섞이지 않도록 별도 함수로 뽑아낸다.
  - 예외 처리를 하는 함수는 `try` 키워드를 함께 사용하는것이 적절

### 반복하지 마라!

- 반복은 가독성을 크게 해친다.
- 동일한 알고리즘(로직)을 사용하는 경우 이를 하나의 함수로 분리해 한 곳에서만 정의되도록 한다.

## 함수를 작성하는 방법

처음에는 길고 복잡해도 된다. 마치 글을 쓰듯 점진적으로 이를 다듬는 방식으로 진행해보자.

1. 함수의 초고(Draft)를 작성한다.
1. 이 함수를 모두 커버하도록 단위 테스트를 작성한다.
1. 위 가이드라인을 따라 함수를 다듬는다. 물론 테스트 역시 모두 통과.
1. 최종적으로 깨끗한 함수가 만들어진다.

이렇게 보면 시스템은 '프로그램'이 아니라 어쩌면 하나의 '글'이라 할 수도 있겠다. 정확한 언어(도메인)로 이야기를 잘 써 내려가보도록 하자.
