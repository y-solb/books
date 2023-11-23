# 6장. 타입 선언과 @types

## 45. devDependencies에 typescript와 @types 추가하기

- `dependencies`

  - 현재 프로젝트를 실행하는 데 필수적인 라이브러리들이 포함된다.
  - 프로젝트를 공개하여 다른 사용자가 해당 프로젝트를 설치한다면 dependencies에 들어 있는 라이브러리도 함께 설치될 것이다.

- `devDependencies`

  - 현재 프로젝트를 개발하고 테스트하는 데 사용되지만, 런타임에는 필요 없는 라이브러리들이 포함된다.
  - 예를 들어, 테스트 프레임워크, 타입스크립트와 관련된 라이브러리
  - 프로젝트를 공개하여 다른 사용자가 해당 프로젝트를 설치한다면 devDependencies에 포함된 라이브러리들은 제외된다는 것이 dependencies와 다른 점이다.

- `peerDependencies`

  - 런타임에 필요하긴 하지만, 의존성을 직접 관리하지 않는 라이브러리들이 포함된다.
  - 단적인 예로 플러그인을 들 수 있다. (제이쿼리 플러그인)

- 타입스크립트를 시스템 레벨로 설치하면 안 된다. 타입스크립트를 프로젝트의 devDependencies에 포함시키고 팀원 모두가 동일한 비전을 사용하도록 해야한다.
- @types 의존성은 dependencies가 아니라 devDependencies에 포함시켜야 한다. 런타임에 @types가 필요한 경우라면 별도의 작업이 필요할 수 있다.

## 46. 타입 선언과 관련된 세 가지 버전 이해하기

- @types 의존성과 관련된 세가지 버전으로 라이브러리 버전, @types 버전, 타입스크립트 버전이 있다.
- 라이브러리를 업데이트 하는 경우 해당 @types 역시 업데이트 해야 한다.
- 타입스크립트로 작성된 라이브러리라면 타입 선언을 자체적으로 포함하고, 자바스크립트로 작성된 라이브러리라면 타입 선언을 DefinitelyTyped에 공개하는 것이 좋다.

## 47. 공개 API에 등장하는 모든 타입을 익스포트하기

- 공개 메서드에 등장한 어떤 형태의 타입이든 익스포트하자. 라이브러리 사용자가 추출할 수 있으므로 익스포트하기 쉽게 만드는 것이 좋다.

## 48. API 주석에 TSDoc 사용하기

- JSDoc/TSDoc 형태를 사용하여 주석을 달면 편집기가 주석 정보를 표시해준다.
- `@param`과 `@returns`구문과 문자 서식을 위해 마크다운을 사용할 수 있다.
- 주석에 타입 정보를 포함하면 안 된다.

```ts
/**
 * 인사말을 생성한다.
 * @param name 인사할 사람의 이름
 * @param title 그 사람의 칭호
 * @returns 인사말
 */
```

## 49. 콜백에서 this에 대한 타입 제공하기

- let이나 const로 선언된 변수가 렉시컬 스코프(lexical scope)인 반면 this는 다이나믹 스코프(dynamic scope)이다. 다이나믹 스코프는 '정의된' 방식이 아닌 '호출된' 방식에 따라 달라진다.

```ts
class C {
  vals = [1, 2, 3];
  logSquares() {
    for (const val of this.vals) {
      console.log(val * val);
    }
  }
}

const c = new C();
c.logSquares();
// 1
// 4
// 9

const c = new C();
const method = c.logSquares;
method(); // Uncaught TypeError: undefined의 'vals' 속성을 읽을 수 없습니다.
// C.prototype.logSquares를 호출하고 this의 값을 c로 바인딩한다.

const c = new C();
const method = c.logSquares;
method.call(c); // 정상
```

```ts
class ResetButton {
  render() {
    return makeButton({ text: "Reset", onClick: this.onClick });
  }
  onClick() {
    alert(`Reset ${this}`);
  }
}
// ResetButton에서 onClick을 호출하면, this 바인딩 문제로 "Reset이 정의되지 않았습니다."라는 경고가 뜬다.

// 생성자 메서드에서 this를 바인딩시키면 해결된다
class ResetButton {
  constructor() {
    this.onClick = this.onClick.bind(this);
  }
  render() {
    return makeButton({ text: "Reset", onClick: this.onClick });
  }
  onClick() {
    alert(`Reset ${this}`);
  }
}
// onClick() {...}은 ResetButton.prototype의 속성을 정의한다.ResetButton의 모든 인스턴스에서 공유된다.
// 생성자에서 this.onClick으로 바인딩하면 onClick속성에 this가 바인딩되어 해당 인스턴스에 생성된다.

// 또는 onClick을 화살표 함수로 바꾸면 해결된다
class ResetButton {
  render() {
    return makeButton({ text: "Reset", onClick: this.onClick });
  }
  onClick = () => {
    alert(`Reset ${this}`); // this가 항상 인스턴스를 참조한다.
  };
}
```

- 콜백 함수에서 this를 사용해야 한다면, 타입 정보를 명시해야 한다.

## 50. 오버로딩 타입보다는 조건부 타입을 사용하기

- 오버로딩 타입보다 조건부 타입을 사용하는 것이 좋다. 조건부 타입은 추가적인 오버로딩 없이 유니온 타입을 지원할 수 있다.

```ts
function double(x) {
  return x + x;
}

// 😰
// - 선언문에는 number타입을 매개변수로 넣고 string을 반환하는 경우도 포함됨
function double(x: number | string): number | string;
const num = double(12); // number | string
const str = double("x"); // number | string

// - 과하게 구체적이다.
// 리터럴 문자열 "x"를 매개변수로 넘긴다고 동일한 리터럴 문자열 "x"타입이 반환되어야 하는 것은 아니다.
function double<T extends number | string>(x: T): T;
const num = double(12); // 타입이 12
const str = double("x"); // 타입이 "x"

// - 여러 가지 타입 선언으로 분리하는 것
function double(x: number): number;
function double(x: string): string;
const num = double(12); // 타입이 number
const str = double("x"); // 타입이 string

function f(x: number | string) {
  return double(x); // 'number | string' 형식의 인수는 'string' 형식의 매개변수에 할당될 수 없습니다.
}

// 👍
// 조건부 타입을 사용하는 것
function double<T extends number | string>(
  x: T
): T extends string ? string : number;

const num = double(12); // 타입이 number
const str = double("x"); // 타입이 string
```

## 51. 의존성 분리를 위해 미러 타입 사용하기

- 필수가 아닌 의존성을 분리할 떄는 구조적 타이핑을 사용하면 된다.

## 52. 테스팅 타입의 함정에 주의하기

- 타입을 테스트할 때는 함수 타입의 동일성과 할당 가능성의 차이점을 알고 있어야 한다.
- 콜백이 있는 함수를 테스트할 때, 콜백 매개변수의 추론된 타입을 체크해야 한다.
- 타입과 관련된 테스트에서 any를 주의해야 한다. 더 엄격한 테스트를 위해 dtslint 같은 도구를 사용하는 것이 좋다.
