# 3장. 타입 추론

## 19. 추론 가능한 타입을 사용해 장황한 코드 방지하기

- 타입스크립트의 많은 구문은 사실 불필요하다. 모든 변수에 타입을 선언하는 것은 비생상적이며 형편없는 스타일이다.
- 이상적인 경우 함수/메서드의 시그니처에는 타입 구문이 있지만, 함수 내의 지역 변수에는 타입 구문이 없다.
- 비구조화 할당문은 모든 지역 변수의 타입이 추론되도록 한다.

```ts
function logProduct(product: Product) {
  const { id, name, price } = product;
  console.log(id, name, price);
}
```

- 함수 매개변수에 기본값이 있는 경우 타입 구문을 생략한다.

```ts
function parseNumber(str: string, base = 10) {
  // ...
}
```

- 타입 정보가 있는 라이브러리에서 콜백 함수의 매개변수 타입은 자동으로 추론된다.

```ts
// 😰
app.get("/health", (request: express.Request, response: express.Response) => {
  response.send("OK");
});

// 👍
app.get("/health", (request, response) => {
  response.send("OK");
});
```

### 타입이 추론될 수 있음에도 타입을 명시하면 좋은 경우

- 객체 리터럴을 정의할 때 타입을 명시하면 잉여 속성 체크가 동작한다. (잉여 속성 체크는 선택적 속성이 있는 타입의 오타 같은 오류를 잡는 데 효과적이다. 변수가 사용되는 순간이 아닌 할당하는 시점에 오류가 표시되도록 해준다.)

```ts
const elmo: Product = {
  name: "Tickle Me Elmo",
  id: "048188 627152",
  price: 28.99,
};

// 타입을 제거한 경우 객체를 선언한 곳이 아니라 객체가 사용되는 곳에서 타입 오류가 발생한다.
const furby = {
  name: "Furby",
  id: 630509430963,
  price: 34,
};
logProduct(furby); // 'id' 속성의 형식이 호환되지 않습니다.
//  'number'형식은 'string'형식에 할당할 수 없습니다.

// 타입 구문을 제대로 명시한다면 실수가 발생한 부분에 오류를 표시한다.
const furby: Product = {
  name: "Furby",
  id: 630509430963, //  'number'형식은 'string'형식에 할당할 수 없습니다.
  price: 34,
};
logProduct(furby);
```

- 함수의 반환에 타입 명시
  - 함수에 대해서 더욱 명확하게 알 수 있다.
  - 명명된 타입을 사용할 수 있다.

```ts
interface Vector2D {
  x: number;
  y: number;
}

function add(a: Vector2D, b: Vector2D) {
  return { x: a.x + b.x, y: a.y + b.y };
}
// 반환 타입을 { x: number; y: number; }으로 추론하지만 Vector2D가 더 직관적인 표현이다.
```

## 20. 다른 타입에는 다른 변수 사용하기

```ts
// 😰
let id = "12-34-56";
fetchProduct(id); // string으로 사용
id = 123456;
fetchProductBySerialNumber(id); // number로 사용

// 👍
const id = "12-34-56";
fetchProduct(id);
const serial = 123456;
fetchProductBySerialNumber(serial);
```

### 다른 타입에 별도의 변수를 사용하는 게 바람직한 이유

- 서로 관련이 없는 두 개의 값을 분리한다.
- 변수명을 더 구체적으로 지을 수 있다.
- 타입 추론을 향상시키며, 타입 구문이 불필요해진다.
- 타입이 좀 더 간결해진다. (`string|number` 대신 `string`과 `number` 사용)
- `let` 대신 `const`로 변수를 선언하게 된다. 타입 체커가 타입을 추론하기에도 좋다.

## 21. 타입 넓히기

- 상수를 사용해서 변수를 초기화할 때 타입을 명시하지 않으면 지정된 단일 값을 가지고 할당 가능한 값들의 집합을 유추해야 한다. 이 과정을 '넓히기(widening)'라고 부른다.

```ts
interface Vector3 {
  x: number;
  y: number;
  z: number;
}
function getComponent(vector: Vector3, axis: "x" | "y" | "z") {
  return vector[axis];
}

let x = "x";
let vec = { x: 10, y: 20, z: 30 };
getComponent(vec, x); // 'string' 형식의 인수는 "x"|"y"|"z" 형식의 매개변수에 할당될 수 없다.
```

### 넓히기의 과정을 제어할 수 있는 방법

- `const`로 변수 선언하기

```ts
const x = "x"; // 타입이 "x"
let vec = { x: 10, y: 20, z: 30 };
getComponent(vec, x); // 이제 x는 재할당될 수 없으므로 👍
```

객체의 경우 타입스크립트의 넓히기 알고리즘은 각 요소를 `let`으로 할당된 것처럼 다룬다.
타입 추론의 강도를 직접 제어하려면 타입스크립트의 기본 동작을 재정의해야 한다.

- 명시적 타입 구문을 제공한다

```ts
const v: { x: 1 | 3 | 5 } = {
  x: 1,
}; // 타입이 { x: 1 | 3 | 5 }
```

- 타입 체커에 추가적인 문맥을 제공한다.
- `const` 단언문을 사용한다
  - `as const`를 작성하면 타입스크립트는 최대한 좁은 타입으로 추론한다.

```ts
const v1 = {
  x: 1,
  y: 2,
}; // 타입은 {x: number; y: number;}

const v2 = {
  x: 1 as const,
  y: 2,
}; // 타입은 {x: 1; y: number;}

const v3 = {
  x: 1,
  y: 2,
} as const; // 타입은 {readonly x: 1; readonly y: 2;}

const a1 = [1, 2, 3]; // 타입이 number[]
const a2 = [1, 2, 3] as const; // 타입이 readonly [1,2,3]
```

## 22. 타입 좁히기

### 타입을 좁히는 방법

- 분기문에서 예외를 던지거나 함수를 반환하여 블록의 나머지 부분에서 변수의 타입을 좁힐 수 있다.

```ts
const el = document.getElementById("foo"); // 타입이 HTMLElement | null
if (!el) throw new Error("Unable to find #foo");
el.innerHTML = "party time"; // 이제 타입은 HTMLElement
```

- `instanceof`

```ts
function contains(text: string, search: string | RegExp) {
  if (search instanceof RegExp) {
    search; // 타입이 RegExp
    return !!search.exec(text);
  }
  search; // 타입이 string
  return text.includes(search);
}
```

- 속성 체크

```ts
interface A {
  a: number;
}
interface B {
  b: number;
}
function pickAB(ab: A | B) {
  if ("a" in ab) {
    ab; // 타입이 A
  } else {
    ab; // 타입이 B
  }
  ab; // 타입이 A|B
}
```

- `Array.isArray`

```ts
function contains(text: string, terms: string | string[]) {
  const termList = Array.isArray(terms) ? terms : [terms];
  termList; // 타입이 string[]
}
```

- 명시적 '태그' 붙이기(태그된 유니온 또는 구별된 유니온)

```ts
interface UploadEvent {
  type: "upload";
  filename: string;
  contents: string;
}
interface DownloadEvent {
  type: "download";
  filename: string;
}
type AppEvent = UploadEvent | DownloadEvent;
function handleEvent(e: AppEvent) {
  switch (e.type) {
    case "download":
      e; // 타입이 DownloadEvent
      break;
    case "upload":
      e; // 타입이 UploadEvent
      break;
  }
}
```

- 사용자 정의 타입 가드

```ts
function isInputElement(el: HTMLElement): el is HTMLInputElement {
  return "value" in el;
}

function getElementContent(el: HTMLElement) {
  if (isInputElement(el)) {
    el; // 타입이 HTMLInputElement
    return el.value;
  }
  el; // 타입이 HTMLElement
  return el.textContent;
}
```

- 유니온 타입에서 `null`을 제외하기 위해 잘못된 방법을 사용한 예제
  - 자바스크립트에서 `typeof null`이 "`object`"이기 때문에 if구문에서 `null`이 제거 되지 않는다.

```ts
const el = document.getElementById("foo"); // 타입이 HTMLElement | null
if (typeof el === "object") {
  el; // 타입이 HTMLElement | null
}
```

    - 빈 문자열 ''과 0 모두 false가 되기 때문에 타입은 좁혀지지 않았다.

```ts
function foo(x?: number | string | null) {
  if (!x) {
    x; // 타입이 string | number | null | undefined
  }
}
```

## 23. 한꺼번에 객체 생성하기

- 여러 속성을 포함해서 한꺼번에 생성해야 타입 추론에 유리하다.

```ts
// 😰
const pt = {};
pt.x = 3; // '{}'형식에 'x'속성이 없습니다.
pt.y = 4; // '{}'형식에 'y'속성이 없습니다.

// 👍
const pt = {
  x: 3,
  y: 4,
}; // 정상
```

- 객체를 반드시 나눠서 만들어야 한다면 타입 단언문(`as`)을 사용해 타입 체커를 통과하게 할 수 있다. (되도록 객체를 한꺼번에 만드는게 좋다!)

```ts
interface Point {
  x: number;
  y: number;
}
const pt = {} as Point;
pt.x = 3;
pt.y = 4; // 정상
```

- 작은 객체들을 조합해서 큰 객체를 만들어야 하는 경우에도 객체 전개 연산자를 사용하면 타입 걱정 없이 필드 단위로 객체를 생성할 수 있다.
- 이때 모든 업데이트마다 새 변수를 사용하여 각각 새로운 타입을 얻도록 하는게 중요하다.

```ts
const pt0 = {};
const pt1 = { ...pt0, x: 3 };
const pt: Point = { ...pt1, y: 4 }; // 정상
```

- 조건부 속성을 추가하려면 속성을 추가하지 않는 `null` 또는 `{}`으로 객체 전개를 사용하면 된다.

```ts
declare let hasMiddle: boolean;
const firstLast = { first: "Harry", last: "Truman" };
const president = { ...firstLast, ...(hasMiddle ? { middle: "S" } : {}) };
// 타입 추론 결과
// const president: {
//   middle?: string;
//   first: string;
//   last: string;
// };
```

- 타입이 유니온으로 추론되는 경우

```ts
declare let hasDates: boolean;
const nameTitle = { name: "Harry", title: "Hello" };
const movie = {
  ...nameTitle,
  ...(hasDates ? { start: -2303, end: 3333 } : {}),
};
// 타입 추론 결과
// const movie: {
//   start: number;
//   end: number;
//   first: string;
//   last: string;
// } |  {
//   first: string;
//   last: string;
// };
```

## 24. 일관성 있는 별칭 사용하기

- 변수에 별칭을 사용할 때는 일관되게 사용해야 한다.
- 비구조화 문법을 사용해서 일관된 이름을 사용하는 것이 좋다.
- 함수 호출이 객체 속성의 타입 정제를 무효화할 수 있다. 속성보다 지역 변수를 사용하면 타입 정제를 믿을 수 있다.

```ts
interface Student {
  name: string;
  age: number;
  major?: string;
}

function checkMajor(student: Student) {
  if (student.major) {
    // ...
  }
}

// 객체 비구조화를 이용해 일관된 이름으로 사용
function checkMajor(student: Student) {
  const { major } = student;
  if (major) {
    // ...
  }
}
```

## 25. 비동기 코드에는 콜백 대신 async 함수 사용하기

- 콜백 지옥(실행 순서는 코드의 순서와 반대, 중첩된 코드로 직관적인 이해가 어려움) -> `promise`(ES2015 도입, `Promise.all`) -> `async await`키워드(ES2017 도입)
- 어떤 함수가 `promise`를 반환한다면 `async`로 선언하는 것이 좋다.

```ts
// fetchURL함수에 캐시를 추가
const _cache: { [url: string]: string } = {};
function fetchWithCache(url: string, callback: (text: string) => void) {
  if (url in _cache) {
    callback(_cache[url]);
  } else {
    fetchURL(url, (text) => {
      _cache[url] = text;
      callback(text);
    });
  }
}

let requestStatus: "loading" | "success" | "error";
function getUser(userId: string) {
  fetchWithCache(`/user/${userId}`, (profile) => {
    requestStatus = "success";
  });
  requestStatus = "loading";
}
// 캐시되어 있지 않다면 requestStatus는 "success"가 된다.
// 캐시되어 있다면 requestStatus는 "success"가 되고 바로 "loading"으로 돌아간다.
```

이를 `async`를 통해 구현하면

```ts
const _cache: { [url: string]: string } = {};
async function fetchWithCache(url: string) {
  if (url in _cache) {
    return _cache[url];
  }
  const response = await fetch(url);
  const text = await response.text();
  _cache[url] = text;
  return text;
}

let requestStatus: "loading" | "success" | "error";
async function getUser(userId: string) {
  requestStatus = "loading";
  const profile = await fetchWithCache(`/user/${userId}`);
  requestStatus = "success";
}
```

## 26. 타입 추론에 문맥이 어떻게 사용되는지 이해하기

- 타입 추론에서 문맥이 어떻게 쓰이는지 알아야 한다.

```ts
type Language = "JavaScript" | "TypeScript" | "Python";

function setLanguage(language: Language) {
  /* ... */
}

setLanguage("JavaScript"); // 정상

let language = "JavaScript";
setLanguage(language); // 'string'형식의 인수는 'Language'형식의 매개변수에 할당될 수 없다.
```

- 해결방법
  - 타입 선언에서 language의 가능한 값을 제한하는 것
  ```ts
  let language: Language = "JavaScript";
  setLanguage(language); // 정상
  ```
  - language를 상수로 만드는 것
  ```ts
  const language = "JavaScript";
  setLanguage(language); // 정상
  ```

### 튜플 사용 시 주의점

```ts
function panTo(where: [number, number]) {
  /* ...*/
}

panTo([10, 20]);

// 😰
const loc = [10, 20];
panTo(loc); // 'number[]' 형식의 인수는 '[number, number]' 형식의 매개변수에 할당될 수 없습니다.

// 👍 - 타입을 선언
const loc: [number, number] = [10, 20]; //
panTo(loc); // 정상

// 👍 - 상수 문맥을 제공(as const)
// 👍 - panTo 함수에 readonly 구문을 추가한다.
function panTo(where: readonly [number, number]) {
  /* ...*/
}
const loc = [10, 20] as const;
panTo(loc); // 정상
```

### 객체 사용 시 주의점

```ts
type Language = "JavaScript" | "TypeScript" | "Python";
interface GovernedLanguage {
  language: Language;
  organization: string;
}

function complain(language: GovernedLanguage) {
  /* ... */
}

complain({ language: "TypeScript", organization: "MS" }); // 정상

const ts = {
  language: "TypeScript",
  organization: "MS",
};

complain(ts); // 'GovernedLanguage'의 형식의 매개변수에 할당될 수 없습니다. 'language' 속성의 형식이 호환되지 않습니다. 'string'형식은 'Language'형식에 할당할 수 없습니다.
```

- 타입 선언을 추가하거나(`const ts:GovernedLanguage = ...`) 상수 단언(`as const`)를 사용하여 해결할 수 있다.

### 콜백 사용 시 주의점

```ts
const fn = (a, b) => {
  console.log(a + b); // 'a' 매개변수에는 암시적으로 'any'형식이 포함된다.
};

callWithRandomNumbers(fn);

// 👍 매개변수에 타입 구문을 추가해서 해결한다.
const fn = (a: number, b: number) => {
  console.log(a + b); // 'a' 매개변수에는 암시적으로 'any'형식이 포함된다.
};

callWithRandomNumbers(fn);
```

## 27. 함수형 기법과 라이브러리로 타입 흐름 유지하기

- 타입 흐름을 개선하고, 가독성을 높이고, 명시적인 타입 구문의 필요성을 줄이기 위해 직접 구현하기보다는 내장된 함수형 기법과 로대시 같은 유틸리티 라이브러리를 사용하는 것이 좋다.
