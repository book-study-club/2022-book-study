# 5. 형식 맞추기

- 코드 형식을 맞추는 것 = 중요 for **가독성**
- 간단한 규칙을 정하고 이를 따르는 것이 좋음
- 팀이 있다면, 그 팀의 규칙에 따르도록 함

## 형식 맞추기 가이드라인 - 세로

### 적절한 행 길이를 유지하라

- 소스 코드를 짧게(200줄 미만) 작성해도 커다란 시스템을 충분히 구축할 수 있다.
- 일반적으로 큰 파일보다 작은 파일이 이해하기 쉽다.

### 신문 기사처럼 작성하라 (위: 고차원, 아래: 저차원)

- 위에서 아래로 내려가며 읽을 수 있도록 작성
- 위에서는 전체적인 요약을, 아래로 내려가며갈수록 그것에 대한 세부 내용을 작성

### 개념은 빈 행으로 분리하라

- 일련의 행 묶음은 어떠한 개념을 표현함
- 이 개념들 사이에는 빈 행을 넣어 분리하는 방식을 이용

```ts
// before
import { curriedAdd, add } from './math';
const add2 = (num: number) => curriedAdd(2)(num);
const multiply = (num: number) => add(num, num);
export default {
  add2,
  multiply,
};
```

```ts
// after
import { curriedAdd, add } from './math';

const add2 = (num: number) => curriedAdd(2)(num);
const multiply = (num: number) => add(num, num);

export default {
  add2,
  multiply,
};
```

### 관련이 있는 행은 서로 가까이 놓아라 (코드 점프를 줄여라)

- 로직을 이해하기 위해 여러 함수나 파일을 옮겨다녀야 한다면 매우 혼란스러워지게 됨
- 서로 관련이 있는 코드는 **한 눈에 들어오도록** 가까이 놓아야 가독성이 높아짐
- 가이드라인
  - 변수: 사용하는 곳에서 최대한 가까운 위치에 선언
  - 인스턴스 변수: 클래스 맨 처음에 선언
  - 종속(호출되는) 함수: 호출하는(caller) 함수를 호출되는(callee) 함수보다 먼저 위치시키고, 서로 가까운 곳에 배치

```ts
// before
export default class {
  /**
   * class name
   */
  #className: string;

  /**
   * log array
   */
  #log: string[];

  #logging(msg: string) {
    this.#log.push(msg);
  }

  /**
   * class properties
   */
  #properties: Property[] = [];
  addProperty(property: Property) {
    this.#properties.push(property);
    this.#logging('add property');
  }
}
```

```ts
// after
export default class {
  #className: string;
  #properties: Property[] = [];
  #log: string[] = [];

  addProperty(property: Property) {
    this.#properties.push(property);
    this.#logging('add property');
  }

  #logging(msg: string) {
    this.#log.push(msg);
  }
}
```

무의미한 주석 또한 제거하는 것이 좋다.

### 개념적 유사성을 띄는 코드는 서로 가까이 배치하라

- 비슷한 동작을 수행하는 경우에도 서로 가까이 배치할 수 있음

  ```ts
  // e.g. Assert function
  export const Assert = {
    assertTrue: (condition: boolean) => {
      if (!condition) {
        fail(condition);
      }
    },
    assertFalse: (condition: boolean) => {
      if (condition) {
        fail(condition);
      }
    },
  };
  ```

## 형식 맞추기 가이드라인 - 가로

### 적절한 행 길이를 유지하라

- 80~100자 정도면 적당
- 가로 정렬은 가독성에 큰 도움이 되지 않았음

  ```ts
  // 가로 정렬 예시
  class Logger {
    #log:           string[]      =    [];
    #systemInfo:    System        =    null;
    #context:       string        =    '';
  }
  ```

- Scope는 들여쓰기를 이용해 반드시 구분

  ```ts
  // bad: 한 줄에 우겨넣지 않는다.
  class Comment {
    constructor(text: string) { super(text); }
    render() { return ''; }
  }
  ```

  ```ts
  // good: 아래와 같이 들여쓰기를 무시하지 않도록 해주자
  class Comment {
    constructor(text: string) {
      super(text);
    }

    render() {
      return '';
    }
  }
  ```
