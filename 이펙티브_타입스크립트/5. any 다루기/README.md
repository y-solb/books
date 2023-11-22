# 5장. any 다루기

- 점진적인 마이그레이션을 할 때 코드의 일부분에 타입 체크를 비활성화시켜주는 any 타입이 중요한 역할을 한다.

## 38. any 타입은 가능한 좁은 범위에서만 사용하기

```ts
function processBar(b: Bar) {
  /* ... */
}

function f() {
  const x = expressionReturningFoo();
  processBar(x);
  // 'Foo' 형식의 인수는 'Bar' 형식의 매개변수에 할당될 수 없다.
}

// 문맥상으로 x라는 변수가 동시에 Foo 타입과 Bar 타입에 할당가능하다면 오류를 제거하는 방법은

// 😰
function f1() {
  const x: any = expressionReturningFoo();
  processBar(x);
}

// 👍
function f2() {
  const x = expressionReturningFoo();
  processBar(x as any); // any 타입이 processBar 매개변수에서만 사용되어 다른 코드에 영향을 미치지 않는다.
}

function f3() {
  const x = expressionReturningFoo();
  // @ts-ignore
  processBar(x); // @ts-ignore를 사용한 다음 줄의 오류가 무시되나 근본적인 원인을 해결한 것은 아니기 때문에 더 큰 문제가 발생할 수 있다.
}
```

```ts
// 😰
const config: Config = {
  a: 1,
  b: 2,
  c: {
    key: value,
  },
} as any;

// 👍
const config: Config = {
  a: 1,
  b: 2,
  c: {
    key: value as any,
  },
};
```

- any의 사용 범위를 최소한으로 좁혀야 한다.
- 함수의 반환 타입이 any인 경우 타입 안전성이 나빠진다. any 타입을 반환하면 절대 안된다.
- 강제로 타입 오류를 제거하려면 any 대신 @ts-ignore 사용하는 것이 좋다.

## 39. any를 구체적으로 변형해서 사용하기

- `any`보다 더 정확하게 모델링할 수 있도록 `any[]` 또는 `{[id:string]: any}` 또는 `()=>any`처럼 구체적인 형태를 사용해야 한다.

## 40. 함수 안으로 타입 단언문 감추기

- 타입 선언문은 일반적으로 타입을 위험하게 만들지만 상황에 따라 필요하기도 하고 현실적인 해결책이 되기도 한다. 불가피하게 사용해야 한다면 정확한 정의를 가지는 함수 안으로 숨기도록 한다.

## 41. any의 진화를 이해하기

- 일반적인 타입들은 정제되기만 하는 반면, 암시적 any와 any[]타입은 진화할 수 있다.
- any를 진화시키는 방식보다 명시적 타입 구문을 사용하는 것이 안전한 타입을 유지하는 방법이다.

```ts
const result = []; // 타입이 any[]
result.push("a"); // 타입이 string[]
result.push(1); // 타입이 (string | number)[]
```

## 42. 모르는 타입의 값에는 any 대신 unknown을 사용하기

- `unknown`은 `any` 대신 사용할 수 있는 안전한 타입이다. 어떠한 값이 있지만 그 타입을 알지 못하는 경우라면 `unknown`을 사용하면 된다.
- 사용자가 타입 단언문이나 타입 체크를 사용하도록 강제하려면 `unknown`을 사용하면 된다.

```ts
// 😰
function parseYAML(yaml: string): any {
  // ...
}

interface Book {
  name: string;
  author: string;
}
const book: Book = parseYAML(`
  name: Wuthering Heights
  author: Emily Bronte
`);

const book = parseYAML(`
  name: Wuthering Heights
  author: Emily Bronte
`);
alert(book.title); // 오류 없음, 런타임에 "undefined" 경고
book("read"); // 오류 없음, 런타임에 "TypeError: book은 함수가 아닙니다" 예외 발생

// 👍
function safeParseYAML(yaml: string): unknown {
  // ...
}

const book = safeParseYAML(`
  name: Wuthering Heights
  author: Emily Bronte
`);
alert(book.title); // ~~ 개체가 'unknown' 형식입니다.
book("read"); //  ~~ 개체가 'unknown' 형식입니다.

// unknown 상태로 사용하려고 하면 오류가 발생하기 때문에, 적절한 타입으로 변환하도록 강제
const book = safeParseYAML(`
  name: Wuthering Heights
  author: Emily Bronte
`) as Book;
alert(book.title); // 'Book' 형식에 'title' 속성이 없습니다.
book("read"); //  이 식은 호출할 수 없습니다.
```

```ts
interface Feature {
  id?: string | number;
  geometry: Geometry;
  properties: unknown;
}

// instanceof를 체크한 후 unknown에서 원하는 타입으로 변환할 수 있다.
function processValue(val: unknown) {
  if (val instanceof Date) {
    val; // 타입이 Date
  }
}

// 사용자 정의 타입 가드
function isBook(val: unknown): val is Book {
  return (
    typeof val === "object" &&
    (val !== null) & ("name" in val) &&
    "author" in val
  );
}
function processValue(val: unknown) {
  if (isBook(val)) {
    val; // 타입이 Book
  }
}

// 제너릭과 타입 단언문을 사용한 것은 기능적으로 동일하나
// 제너릭보다는 unknown을 반환하고 사용자가 직접 단언문을 사용하거나 원하는 대로 타입을 좁히도록 강제하는 것이 좋다.
function safeParseYAML<T>(yaml: string): T {
  // ...
}
```

## 43. 몽키 패치보다는 안전한 타입을 사용하기

- 객체에 임의의 속성을 추가하는 것은 일반적으로 좋은 설계가 아니다.
  - 예를 들어 window 또는 DOM 노드에 데이터를 추가한다고 가정하면 그 데이터는 전역 변수가 된다. 전역변수를 사용하면 서로 멀리 떨어진 부분들 간에 의존성을 만들게 된다.

```ts
document.monkey = "Hello"; // ~~ 'Document' 유형에 'monkey' 속성이 없습니다.

// any 단언문으로 해결
// - 타입의 안전성을 상실하고 언어 서비스를 사용할 수 없다
(document as any).monkey = "Hello"; // 정상

// interface의 보강(augmentation)을 사용
// - 타입이 안전하고 타입 체커는 오타나 잘못된 타입의 할당을 오류로 표시한다.
// - 속성에 주석을 붙이고 자동완성을 사용할 수 있다.
interface Document {
  monkey: string;
}

document.monkey = "Hello"; // 정상

// 모듈의 관점에서(import/export를 사용하는 경우) global 선언을 추가해야 한다.
export {};
declare global {
  interface Document {
    monkey: string;
  }
}
document.monkey = "Hello"; // 정상

// 더 구체적인 타입 단언문을 사용해 해결
// - MonkeyDocument는 Document를 확장하기 때문에 타입 단언문은 정상이며 할당문의 타입은 안전하다.
// - Document 타입을 건드리지 않고 별도로 확장하는 새로운 타입을 도입했기 때문에 모듈 영역 문제도 해결할 수 있다.
interface MonkeyDocument extends Document {
  monkey: string;
}
(document as MonkeyDocument).monkey = "Hello";
```

## 44. 타입 커버리지를 추적하여 타입 안전성 유지하기

- noImplicitAny가 설정되어 있어도 명시적 any 또는 서드파티 타입 선언(@types)을 통해 any 타입은 코드 내에 존재할 수 있다는 점을 주의해야 한다.
- npm의 `type-cover-age`(npx type-cover-age)패키지를 활용하여 any를 추적하면 좋다.
