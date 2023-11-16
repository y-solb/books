# 2장. 타입스크립트의 타입 시스템

## 6. 편집기를 사용하여 타입시스템 탐색하기

- 타입스크립트를 설치하면 아래 두가지를 실행할 수 있다.

  - 타입스크립트 컴파일러(tsc)

  - 단독으로 실행할 수 있는 타입스크립트 서버(tsserver)
    -> 코드 자동완성, 명세 검사, 검색, 리팩토링이 포함

- 편집기를 사용하면 어떻게 동작하고 타입을 추론하는지 알 수 있다.
- 타입스크립트가 동작을 어떻게 모델링하는지 알기 위해 타입 선언 파일을 찾아보면 된다.

## 7. 타입을 값들의 집합이라고 생각하기

- 타입스크립트 타입은 엄격한 상속 관계가 아니라 겹쳐지는 집합으로 표현된다.

## 8. 타입 공간과 값 공간의 심벌 구분하기

- 타입스크립트 코드에서 타입인지 값인지 구분할 수 있어야 한다.

## 9. 타입 단언보다는 타입 선언을 이용하기

- 타입 단언은 강제로 타입을 지정했으니 타입 체커에게 오류를 무시하라고 하는 것이다.

```ts
interface Person {
  name: string;
}

const alice: Person = {};
// 'Person' 유형에 필요한 'name'속성이 '{}'유형에 없습니다.
const bob = {} as Person; // 오류 없음(오류를 무시함)

const people = ["alice", "bob", "jan"].map((name) => ({ name }));
// Person[]을 원했지만 결과는 {name:string;}[]...

// 올바른 사용
// - 원하는 타입을 직접 명시하기
const people: Person[] = ["alice", "bob", "jan"].map(
  (name): Person => ({ name })
);
```

### 타입 단언이 꼭 필요한 경우

- 타입스크립트는 DOM에 접근할 수 없기 때문에 `#myButton`이 버튼 엘리먼트인지 알지 못한다.

```ts
document.querySelector("#myButton").addEventListener("click", (e) => {
  const button = e.currentTarget as HTMLButtonElement; // 타입은 HTMLButtonElement
});
```

- `null`이 아님을 단언하는 경우(!)(또는 `null`인 경우를 체크하는 조건문을 사용해야 한다.)

```ts
const elNull = document.getElementById("foo"); // 타입은 HTMLElement | null
const el = document.getElementById("foo")!; // 타입은 HTMLElement
```

## 10. 객체 래퍼 타입 피하기

- `string` '기본형'에는 메서드가 없지만 자바스크립트에는 메서드를 가지는 `String` '객체'타입이 정의되어 있다.
- `string` 기본형에 `chatAt` 같은 메서드를 사용할 때, 자바스크립트는 기본형을 `String` 객체로 래핑하고, 메서드를 호출하고, 마지막에 래핑한 객체를 버린다.
- 타입스크립트 객체 래퍼 타입은 지양하고, 대신 기본형 타입을 사용해야 한다.

## 11. 잉여 속성 체크의 한계 인지하기

- 구조적 타입 시스템에서 발생할 수 있는 오류를 잡을 수 있도록 '잉여 속성 체크'라는 과정이 수행된다.
- 객체 리터럴을 변수에 할당하거나 함수에 매개변수로 전달할 때 잉여 속성 체크가 수행된다.

```ts
interface Room {
  numDoors: number;
  ceilingHeightFt: number;
}

const r :Room {
  numDoors:1,
  ceilingHeightFt: 10,
  elephant:'present' // 개체 리터럴은 알려진 속성만 지정할 수 있으며 'Room'형식에 'elephant'가 없습니다.
}
```

```ts
interface Options {
  title: string;
  darkMode?: boolean;
}

function createWindow(options: Options) {
  if (options.darkMode) {
    ...
  }
}

// 잉여 속성 체크
createWindow({
  title: 'Spider Solitarire',
  darkmode: true // 개체 리터럴은 알려진 속성만 지정할 수 있지만 'Options'형식에 'darkmode'가 없습니다. 'darkMode'를 쓰려고 했습니까?
})

// document와 new HTMLAnchorElement는 객체 리터럴이 아니기 때문에 잉여 속성 체크가 되지 않는다.
const o1:Options = document; // 정상
const o2:Options = new HTMLAnchorElement; // 정상
```

한계점

- 임시 변수를 도입하면 잉영 속성 체크를 건너뛸 수 있다
- 잉여 속성 체크는 타입 단언문을 사용할 때에도 적용되지 않는다.

```ts
const o = { darkmode: true, title: "Hello world" } as Options; // 정상
```

## 12. 함수 표현식에 타입 적용하기

- 자바스크립트와 타입스크립트에서는 함수 '문장'과 함수 '표현식'을 다르게 인식한다.

```ts
function rollDice1(sides: number): number {
  /* ... */
} // 문장
const rollDice2 = function (sides: number): number {
  /* ... */
}; // 표현식
const rollDice3 = (sides: number): number {
  /* ... */
} // 표현식
```

- 타입스크립트에서는 함수 표현식을 사용하는 것이 좋다. 함수의 매개변수부터 반환값까지 전체를 함수 타입으로 선언하여 함수 표현식에 재사용할 수 있다.

```ts
type DiceRollFn = (sides: number) => number;
const rollDice: DiceRollFn = (sides) => {
  /* ... */
};
```

- 다른 함수의 시그니처를 참조하려면 `typeof fn`을 사용하면 된다.

## 13. 타입과 인터페이스의 차이점 알기

### 타입과 인터페이스의 공통점

```ts
// 인덱스 시그니처
type TDict = { [key: string]: string };

interface IDict {
  [key: string]: string;
}

// 함수 타입
type TFn = (x: number) => string;

interface IFn {
  (x: number): string;
}

// 제너릭
type TPair<T> {
  first: T;
  second: T;
}

interface IPair<T> {
  first: T;
  second: T;
}

// 인터페이스는 타입을 확장할 수 있다. (주의사항: 유니온 타입 같은 복잡한 타입은 확장하지 못한다.)
interface IStateWithPop extends TState {
  population: number;
}

// 타입은 인터페이스를 확장할 수 있다.
type TStateWithPop = IState & {
  population: number;
}
```

### 타입과 인터페이스의 차이점

- 유니온 타입은 있지만 유니온 인터페이스는 없다.

```ts
type AorB = "a" | "b";
```

- 인터페이스는 타입을 확장할 수 있지만, 유니온은 할 수 없다. 그런데 유니온 타입을 확장하는게 필요할 때가 있다.

```ts
type Input { /* ... */ }
type Output { /* ... */ }
interface VariableMap{
  [name: string]: Input | Output;
}
```

- 유니온 타입에 name 속성을 붙인 타입을 만들 수 있다 (인터페이스로 표현은 불가능)

```ts
type NamedVariable = (Input | Output) & { name: string };
```

- 튜플과 배열 타입도 type 키워드를 이용해 간결하게 표현할 수 있다.

```ts
type Pair = [number, number];
type StringList = string[];
type NamedNums = [string, ...number[]];
```

(인터페이스로 튜플과 비슷하게 구현)

- 인터페이스로 튜플과 비슷하게 구현하면 튜플에서 사용할 수 있는 `concat` 같은 메서드들을 사용할 수 없다.
- 튜플은 `type` 키워드로 구현하는게 낫다.

```ts
interface Tuple {
  0: number;
  1: number;
  length: 2;
}
const t: Tuple = [10, 20];
```

- 인터페이스에서는 타입에 없는 보강(augment)이 가능하다.

```ts
// 선언 병합
interface IState {
  name: string;
  capital: string;
}

interface IState {
  population: number;
}

const chursu:IState = {
  name: 'Chursu';
  capital: 'Seoul';
  population: 500000
}
```

## 14. 타입 연산과 제너릭 사용으로 반복 줄이기

```ts
// 타입에 이름 붙이기
function distance(
  a: {
    x: number;
    y: number;
  },
  b: {
    x: number;
    y: number;
  }
) {
  /* ... */
}

interface Point2D {
  x: number;
  y: number;
}

function distance(a: Point2D, b: Point2D) {
  /* ... */
}

// 시그니처를 명명된 타입으로 분리
const get: (url: string, opts: Options):Promise<Response>  = (url, opts) => {
  /* ... */
};
const post: (url: string, opts: Options):Promise<Response>  = (url, opts) => {
  /* ... */
};

type HTTPFunc = (url: string, opts: Options) => Promise<Response>;
const get: HTTPFunc = (url, opts) => {
  /* ... */
};
const post: HTTPFunc = (url, opts) => {
  /* ... */
};

// 인터페이스 확장
interface Person {
  firstName: string;
  lastName: string;
}

interface PersonWithBirthDate extends Person {
  birth: Date;
}

// 타입 확장
type PersonWithBirthDaten= Person & { birth: Date }
```

- `type Pick<T,K> = {[k in K]: T[k]}`

```ts
interface State {
  userId: string;
  pageTitle: string;
  recentFiles: string[];
  pageContents: string;
}

interface TopNavState {
  userId: string;
  pageTitle: string;
  recentFiles: string[];
}

// State의 부분집합으로 TopNavState를 정의(State내의 pageTitle의 타입이 변경되면 같이 반영됨)
type TopNavState {
  userId: State['userId'];
  pageTitle: State['pageTitle'];
  recentFiles: State['recentFiles'];
}

type TopNavState = {
  [k in 'userId' | 'pageTitle' | 'recentFiles']: State[k]
}

type TopNavState = Pick<State, 'userId' | 'pageTitle' | 'recentFiles'>
```

```ts
interface SaveAction {
  type: "save";
  // ...
}

interface LoadAction {
  type: "load";
  // ...
}

type Action = SaveAction | LoadAction;
type ActionType = "save" | "load";

// Action 유니온을 인덱싱
type ActionType = Action["type"]; // "save" | "load"

// Pick을 사용하여 얻게 되는 것과 다름
type ActionRed = Pick<Action, "type">; // {type: "save" | "load"}
```

- `Partial` 매핑된 타입(`[key in keyof Options]`)을 순회하며 Options 내 k 값에 해당하는 속성이 있는지 찾고 ?는 각 속성을 선택적으로 만든다.

```ts
interface Options {
  width: number;
  height: number;
  color: string;
  label: string;
}

interface OptionsUpdate {
  width?: number;
  height?: number;
  color?: string;
  label?: string;
}
// 매핑된 타입과 keyof를 사용
type OptionsUpdate = { [key in keyof Options]?: Options[k] };
// Partial
type OptionsUpdate = Partial<Options>;
```

- 값의 형태에 해당하는 타입을 정의하고 싶을 때

```ts
const INIT_OPTIONS = {
  width: 640,
  height: 480,
  color: "#FE5000",
  label: "VGA",
};

interface Options {
  width: number;
  height: number;
  color: string;
  label: string;
}
// typeof(타입스크립트에서 연산됨)
type Options = typeof INIT_OPTIONS;
```

- `ReturnType`

```ts
function getUserInfo(userId: string) {
  // ...
  return {
    userId, name,age,...
  }
}
type UserInfo = ReturnType<typeof getUserInfo>
```

## 15. 동적 데이터에 인덱스 시그니처 사용하기

- 타입에 '인덱스 시그니처'를 명시하여 유연하게 매핑을 표현할 수 있다.

```ts
type Rocket = { [property: string]: string };

const rocket: Rocket = {
  name: "Falcon 9",
  variant: "v1.0",
  thrust: "4940kN",
};
```

### 인덱스 시그니처 사용 시 단점

- 잘못된 키를 포함해 모든 키를 허용한다.
- 특정 키가 필요하지 않다. {}도 유효한 Rocket 타입이다.
- 키마다 다른 타입을 가질 수 없다.
- 자동 완성 기능이 동작하지 않는다.

인덱스 시그니처는 동적 데이터를 표현할 때 사용한다

- 런타임 때까지 객체의 속성을 알 수 없을 경우 (CSV 파일에서 로드하는 경우)
- 안전한 접근을 위해 인덱스 시그니처의 값 타입에 undefined를 추가하는 것을 고려해야 한다.

가능한 인터페이스, Record, 매핑된 타입 같은 인덱스 시그니처보다 정확한 타입을 사용하는 것이 좋다.

```ts
interface Row1 {[column: string]: number} // 너무 광범위
interface Row2 {a: number; b?: number; c?: number} // 최선
type Row3 =  {a: number} | {a: number; b: number;} |  {a: number; b: number; c: number;} // 가장 정확하지만 사용하기 번거로움

// Record는 키 타입에 유연성을 제공하는 제너릭 타입이다.
type Vec3D = Record<"x" | "y" | "z", number>;
// Type Vec3D ={
//   x: number;
//   y: number;
//   z: number;
// }

// 매핑된 타입을 사용해 키마다 별도의 타입을 사용하게 해준다.
type Vec3D {[k in "x" | "y" | "z"]: number};
// Type Vec3D ={
//   x: number;
//   y: number;
//   z: number;
// }
type ABC {[k in "a" | "b" | "c"]: k extends 'b' ? string : number};
// Type ABC ={
//   x: number;
//   y: string;
//   z: number;
// }
```

## 16. number 인덱스 시그니처보다는 Array, 튜플, ArrayLike를 사용하기

- 배열은 객체이므로 키는 숫자가 아니라 문자열이다.
- 인덱스 시그니처에 `number`를 사용하기보다 Array나 튜플 또는 ArrayLike타입을 사용하는 것이 좋다.

## 17. 변경 관련된 오류 방지를 위해 readonly 사용하기

- 함수가 매개변수를 수정하지 않는다면 `readonly`로 선언하는 것이 좋다. 이는 인터페이스를 명확하게 하며, 매개변수가 변경되는 것을 방지한다.
- `const`와 `readonly`의 차이를 이해해야 한다.
- `readonly`는 얕게 동작한다.

## 18. 매핑된 타입을 사용하여 값을 동기화하기

- 매핑된 타입을 사용해서 관련된 값과 타입을 동기화하도록 한다.
- 인터페이스에 새로운 속성을 추가할 때, 선택을 강제하도록 매핑된 타입을 고려해야 한다.
