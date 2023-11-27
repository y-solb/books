# 8장. 타입스크립트로 마이그레이션하기

## 58. 모던 자바스크립트로 작성하기

- 타입스크립트는 자바스크립트의 상위집합이기 때문에, 최신 버전의 자바스크립트 코드를 옛날 버전의 자바스크립트 코드로 변환할 수 있다.

- ECMAScript 모듈(import/export) 사용하기

- 프로토타입 대신 클래스 사용하기

- var 대신 let/const 사용하기

```ts
function foo() {
  bar();
  function bar() {
    console.log("hello");
  }
}
```

foo 함수를 호출하면 bar 함수의 정의가 호이스팅 되어 가장 먼저 수행되기 때문에 bar함수가 문제없이 호출되고 hello가 출력된다. 호이스팅은 실행 순서를 예상하기 어렵게 만들고 직관적이지 않다. 대신 함수 표현식(const bar = ()=>{...})을 사용하여 호이스팅 문제를 피하는 것이 좋다.

- for(;;;) 대신 for-of 또는 배열 메서드 사용하기

- 함수 표현식 보다 화살표 함수 사용하기

```ts
class Foo {
  method() {
    console.log(this);
    [1, 2].forEach(function (i) {
      console.log(this);
    });
  }
}

const f = new Foo();
f.method();
// strict 모드에서 Foo, undefined, undefined
// non-strict 모드에서 Foo, window, window
```

화살표 함수를 사용하면 상위 스코프의 this를 유지할 수 있다.

```ts
class Foo {
  method() {
    console.log(this);
    [1, 2].forEach((i) => {
      console.log(this);
    });
  }
}

const f = new Foo();
f.method();
// 항상 Foo, Foo, Foo
```

noImplicitThis를 설정하면 타입스크립트가 this 바인딩 관련된 오류를 표시해 주므로 설정하는 것이 좋다.

- 단축 객체 표현과 구조 분해 할당 사용하기

```ts
const x = 1,
  y = 2,
  z = 3;
const pt = { x, y, z };

["A", "B", "C"].map((char, idx) => ({ char, idx }));
// [ {char: 'A', idx: 0}, {char: 'B', idx: 1}, {char: 'C', idx: 2} ]

const props = obj.props;
const a = props.a;
const b = props.b;

const { props } = obj;
const { a, b } = props;

const {
  props: { a, b },
} = obj; // props는 변수 선언이 아님

// 기본값 지정도 가능
let { a } = obj.props;
if (a === undefined) a = "default";

const { a = "default" } = obj.props;

// 배열에도 구조 분해 문법 사용 가능
const point = [1, 2, 3];
const [x, y, z] = point;
const [, a, b] = point; // 첫번째 요소 무시

// 함수 매개변수에도 구조 분해 문법 사용 가능
const points = [
  [1, 2, 3],
  [4, 5, 6],
];
points.forEach(([x, y, z]) => console.log(x + y + z));
```

- 함수 매개변수 기본값 사용하기
- 저수준 프로미스나 콜백 대신 async/await 사용하기
- 연관 배열에 객체 대신 Map과 Set 사용하기
- 타입스크립트에 use strict 넣지 않기

## 59. 타입스크립트 도입 전에 @ts-check와 JSDoc으로 시험해보기

- 파일 상단에 // @ts-check를 추가하면 자바스크립트에서도 타입 체크를 수행할 수 있다.
- JSDoc 주석을 잘 활용하면 자바스크립트 상태에서도 타입 단언과 타입 추론을 할 수 있다.

## 60. allowJs로 타입스크립트와 자바스크립트 같이 사용하기

- 점진적 마이그레이션을 위해 자바스크립트와 타입스크립트를 동시에 사용할 수 있게 allowJs 컴파일러 옵션을 사용하자.
- 대규모 마이그레이션 작업을 시작하기 전에 테스트와 빌드 체인에 타입스크립트를 적용해야 한다.

## 61. 의존성 관계에 따라 모듈 단위로 전환하기

- 마이그레이션의 첫 단계는, 서드파티 모듈과 외부 API 호출에 대한 @types를 추가하는 것이다.
- 의존성 관계도의 아래부터 위로 올라가며 마이그레이션을 하면 된다. 첫번째 모듈은 보통 유틸리티 모듈이다.
- 이상한 설계를 발견하더라도 리팩토링을 하면 안된다.
- 타입스크립트로 전환하며 발견하게 되는 일반적인 오류들을 놓치지 않고 타입 정보를 유지하기 위해 필요에 따라 JSDoc 주석을 활용해야 할 수도 있다.

## 62. 마이그레이션의 완성을 위해 noImplicitAny 설정하기

- `noImplicitAny` 설정이 없다면 타입 선언과 관련된 실제 오류가 드러나지 않는다.
