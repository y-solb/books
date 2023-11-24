# 7장. 코드를 작성하고 실행하기

## 53. 타입스크립트 기능보다 ECMAScript 기능을 사용하기

### 열거형(enum)

```ts
enum Flavor {
  VALILLA = 0,
  CHOCOLATE = 1,
  STRAWBERRY = 2,
}

let flavor = Flavor.CHOCOLATE; // 타입이 Flavor
```

타입스크립트의 열거형 문제점

- 숫자 열거형에 0,1,2외의 다른 숫자가 할당되면 매우 위험하다.
- 상수 열거형은 보통의 열거형과 달리 런타임에 완전히 제거된다. 앞의 예제를 const enum Flavor로 바꾸면, 컴파일러는 Flavor.CHOCOLATE을 0으로 바꿔버린다. 이런 결과는 기대하지 않은 것이며, 문자열 열거형과 숫자 열거형과 전혀 다른 동작이다.
- preserveConstEnums플래그를 설정한 상태의 상수 열거형은 보통 열거형처럼 런타임 코드에 상수 열거형 정보를 유지한다.
- 문자열 열거형은 런타임의 타입 안전성과 투명성을 제공한다. 그러나 타입스크립트의 다른 타입과 달리 구조적 타이핑이 아닌 명목적 타이핑을 사용한다.

```ts
enum Flavor {
  VALILLA = "vanilla",
  CHOCOLATE = "chocolate",
  STRAWBERRY = "strawberry",
}
let flavor = Flavor.CHOCOLATE; // 타입이 Flavor
flavor = "strawberry"; // 'strawberry'형식은 'Flavor'형식에 할당될 수 없습니다.
function scoop(flavor: Flavor) {
  /* ... */
}

scoop("vanilla"); // 자바스크립트에서 정상

scoop("vanilla"); // 타입스크립트 'vanilla'형식은 'Flavor'형식의 매개변수에 할당될 수 없습니다.
```

- 자바스크립트와 타입스크립트에서 동작이 다르기 떄문에 문자열 열거형은 사용하지 않는 것이 좋다.
- 열거형 대신 리터럴 타입의 유니온을 사용하는 것이 좋다.

```ts
type Flavor = "vanilla" | "chocolate" | "strawberry";

let flavor: Flavor = "chocolate"; // 정상
flavor = "mint chip"; // "mint chip"유형은 'Flavor'유형에 할당될 수 없습니다.
```

### 매개변수 속성

```ts
// 일반적으로 클래스 초기화할 때 속성을 할당하기 위해 생성자의 매개변수를 사용한다
class Person {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
}

// 타입스크립트에서는 더 간결한 문법을 제공한다.(public name: 매개변수 속성)
class Person {
  constructor(public name: string) {}
}
```

매개변수 속성 문제점

- 일반적으로 타입스크립트 컴파일은 타입 제거가 이루어지므로 코드가 줄어들지만, 매개변수 속성은 코드가 늘어나는 문법이다.
- 매개변수 속성이 런타임에는 실제로 사용되지만, 타입스크립트 관점에서는 사용되지 않는 것처럼 보인다.
- 매개변수 속성과 일반 속성을 섞어서 사용하면 클래스의 설계가 혼란스러워진다.

### 네임스페이스와 트리플 슬래시 임포트

- 트리플 슬래시 임포트와 module 키워드는 호환성을 위해 남아 있을 뿐이며, 이제 ECMAScript 2015 스타일의 모듈(import와 export)을 사용해야 한다.

### 데코레이터

- 데코레이터는 클래스, 메서드, 속성에 애너테이션을 붙이거나 기능을 추가하는 데 사용한다. 앵귤러 프레임워크를 지원하기 위해 추가되었으며 표준화가 완료되지 않았기 때문에 사용하지 않는 게 좋다.

## 54. 객체를 순회하는 노하우

```ts
const obj = {
  one: "uno",
  two: "dos",
  three: "tres",
};

for (const k in obj) {
  const v = obj[k]; // ~~ obj에 인덱스 시그니처가 없기 때문에 엘리먼트는 암시적으로 'any' 타입입니다.
}
// 오류의 원인은 k의 타입은 string이지만 obj객체에는 'one','two','three' 세 개의 키만 존재하기 때문이다.
```

- keyof로 해결
  - 상수이거나 추가적인 키없이 정확한 타입을 원하는 경우에 적절하다.

```ts
let k: keyof typeof obj; // 'one','two','three' 타입
for (k in obj) {
  const v = obj[k]; // 정상
}

interface ABC {
  a: string;
  b: string;
  c: number;
}

function foo(abc: ABC) {
  for (const k in abc) {
    const v = abc[k]; // 'ABC'타입에 인덱스 시그니처가 없기 때문에 엘리먼트는 암시적으로 'any' 타입입니다.
  }
}

const x = { a: "a", b: "b", c: 2, d: new Date() };
foo(x); // 정상

// ABC 타입에 할당 가능한 객체는 a,b,c 외에 다른 속성이 존재할 수 있기 때문에, 타입스크립트는 ABC 타입의 키를 string으로 선택해야 한다.
function foo(abc: ABC) {
  let k: keyof ABC;
  for (const k in abc) {
    // let k: "a"|"b"|"c"
    const v = abc[k]; // string|number 타입
  }
}
// v도 string|number 타입으로 한정되어 범위가 너무 좁아 문제가 된다.
// d: new Date()가 있을 수 있어 d 속성은 Date 타입뿐만 아니라 어떠한 타입이든 될 수 있기 때문에 v가 string|number 타입으로 추론된 것은 잘못이며 런타임의 동작을 예상하기 어렵다.
```

- Object.entries로 해결
  - 단지 객체의 키와 값을 순회하고 싶을 때 사용한다.

```ts
function foo(abc: ABC) {
  for (const [k, v] of Object.entries(abc)) {
    k; // string 타입
    v; // any 타입
  }
}
```

## 55. DOM 계층 구조 이해하기

- EventTarget

  - DOM 타입 중 가장 추상화된 타입이다.
  - window, XMLHttpRequest

- Node

  -
  - document, Text, Comment

- Element

  - HTMLElement, SVGElement 포함

- HTMLElement

  - <i>, <b>

- HTMLButtonElement

  - <button>

- Event
  - UIEvent: 모든 종류의 사용자 인터페이스 이벤트
  - MouseEvent: 클릭처럼 마우스로부터 발생되는 이벤트
  - TouchEvent: 모바일 기기의 터치 이벤트
  - WheelEvent: 스크롤 휠을 돌려서 발생되는 이벤트
  - keyboardEvent: 키 누름 이벤트

## 56. 정보를 감추는 목적으로 private 사용하지 않기

```ts
class Diary {
  private secret = "today is ...";
}

const diary = new Diary();
diary.secret; // 'secret'속성은 private이며 'Diary'내에서만 접근할 수 있습니다.

(diary as any).secret; // 😰 정상
```

- 타입스크립트에는 public, protected, private 접근 제어자를 사용해서 공개 규칙을 강제할 수 있는 것으로 오해할 수 있다. public, protected, private 같은 접근 제어자는 타입스크립트 키워드이기 때문에 컴파일 후에는 제거된다. 타입스크립트 접근 제어자들은 단지 컴파일 시점에만 오류를 표시해 줄 뿐이며, 런타임에는 아무런 효력이 없다. 심지어 단언문을 사용하면 타입스크립트 상태에서도 private 속성에 접근할 수 있다.

- 자바스크립트에서 정보를 숨기기 위해 가장 효과적인 방법은 클로저를 사용하는 것이다.
- passwordHash를 생성자 외부에서 접근할 수 없기 떄문에, passwordHash에 접근해야 하는 메서드 역시 생성자 내부에 정의되어야 한다.
- 메서드 정의가 생성자 내부에 존재하게 되면 인스턴스를 생성할 때마다 각 메서드의 복사본이 생성되기 때문에 메모리를 낭비하게 된다는 것을 기억해야 한다.

```ts
declare function hash(text: string): number;
class PasswordChecker {
  checkPassword: (password: string) => boolean;
  constructor(passwordHash: number) {
    this.checkPassword = (password: string) => {
      return hash(password) === passwordHash;
    };
  }
}
```

- 비공개 필드 기능으로 접두사로 #를 붙여서 타입 체크와 런타임 모두에서 비공개로 만듬(현재 표준화가 진행중)
- 클로저와 다르게 클래스 메서드나 동일한 클래스의 개별 인스턴스끼리는 접근이 가능하다.

```ts
class PasswordChecker {
  #passwordHash: number;

  constructor(passwordHash: number) {
    this.#passwordHash = passwordHash;
  }

  checkPassword(password: string) {
    return hash(password) === this.#passwordHash;
  }
}

const checker = new PasswordChecker(hash("s3cret"));
checker.checkPassword("secret"); // 결과는 false
checker.checkPassword("s3cret"); // 결과는 true
```

## 57. 소스맵을 사용하여 타입스크립트 디버깅하기

- 원본 코드가 아닌 변환된 자바스크립트 코드를 디버깅하지 맙자. 소스맵을 사용해서 런타임에 타입스크립트 코드를 디버깅하자.
- 소스맵이 최종적으로 변환된 코드에 완전히 매핑되어있는지 확인하자.
- 소스맵에 원본 코드가 그대로 포함되도록 설정되어 있을 수도 있다. 공개되지 않도록 설정을 확인하자.
