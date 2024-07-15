# 9장. 타입 변환과 단축 평가

## 9.1 타입 변환이란?

- 명시적 타입 변환: 개발자가 의도적으로 값의 타입을 변환하는 것

  ```js
  var x = 10;

  var str = x.toString();
  console.log(typeof str, str); // string 10
  ```

- 암묵적 타입 변환: 개발자의 의도와는 상관없이 표현식을 평가하는 도중에 자바스크립트 엔진에 의해 암묵적으로 타입이 자동 변환되는 것

  ```js
  var x = 10;

  var str = x + "";
  console.log(typeof str, str); // string 10
  ```

## 9.3 명시적 타입 변환

### 9.3.3 불리언 타입으로 변환

```js
// ! 부정 논리 연산자를 두번 사용하는 방법
// 문자열 타입 -> 불리언 타입
!!"x"; // true
!!""; // false
// 숫자 타입 -> 불리언 타입
!!0; // false
!!1; // true
```

## 9.4 단축 평가

### 9.4.1 논리 연산자를 사용한 단축 평가

논리 연산의 결과를 결정하는 피연산자를 타입 변환하지 않고 그대로 반환한다. 단축 평가는 표현식을 평가하는 도중에 평가 결과가 확정된 경우 나머지 평가 과정을 생략하는 것을 말한다.

```js
"Cat" && "Dog"; // Dog
"Cat" || "Dog"; // Cat
```

- 단축평가가 유용하게 사용되는 경우

  - 객체를 가리키기를 기대하는 변수가 `null` 또는 `undefined`가 아닌지 확인하고 프로퍼티를 참조할때
    ```js
    var elem = null;
    var value = elem.value; // TypeError: Cannot read property 'value' of null
    var value = elem && elem.value; // null
    ```
  - 함수 매개변수에 기본값을 설정할 때

    ```js
    function getStringLength(str) {
      str = str || "";
      return str.length;
    }

    getStringLength(); // 0
    getStringLength("hi"); // 2
    ```

### 9.4.2 옵셔널 체이닝 연산자

좌항의 피연산자가 `null` 또는 `undefined`인 경우 `undefined`를 반환하고, 그렇지 않으면 우항의 프로퍼티 참조를 이어간다.

```js
var elem = null;
var value = elem?.value;
console.log(value); // undefined
```

옵셔널 체이닝 연산자 `?.`가 도입되기 이전에는 논리 연산자 &&를 사용한 단축 평가를 통해 변수가 `null` 또는 `undefined`인지 확인했다.

```js
var elem = null;
// Falsy값(false, undefined, null, 0, -0, NaN, '')이면 좌항 피연산자를 그대로 반환한다.
var value = elem && elem.value;
console.log(value); // null
```

### 9.4.3 null 병합 연산자

null 병합 연산자 `??`는 좌항의 피연산자가 `null` 또는 `undefined`인 경우 우항의 피연산자를 반환하고, 그렇지 않으면 좌항의 피연산자를 반환한다. 변수에 기본값을 설정할 때 유용하다.

```js
var foo = null ?? "default string";
console.log(foo); // default string
```

```js
// Falsy값이나 0 또는 ''도 기본값으로서 유효하다면 예기치 않은 동작이 발생할 수 있다.
var foo = "" || "default string";
console.log(foo); // default string

// 좌항의 피연산자가 Falsy값이라도 null 또는 undefined가 아니면 좌항의 피연산자를 반환한다.
var foo = "" ?? "default string";
console.log(foo); // ''
```
