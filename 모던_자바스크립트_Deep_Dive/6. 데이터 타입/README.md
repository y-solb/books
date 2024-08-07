# 6장. 데이터 타입

- 원시타입: number, string, boolean, undefined, null, symbol
- 객체타입: 객체, 함수, 배열 등

## 6.5 undefined 타입

- undefined는 개발자가 의도적으로 할당하기 위한 값이 아니라 자바스크립트 엔진이 변수를 초기화할 때 사용하는 값이다.
- null은 변수에 값이 없다는 것을 명시하고 싶을 때 사용한다.

## 6.10 동적 타이핑

- 정적 타입 언어(C,C++,JAVA...)는 컴파일 시점에 타입 체크(선언한 데이터 타입에 맞는 값을 할당했는지 검사하는 처리)를 수행한다. 정적 타입 언어는 변수 선언 시점에 변수의 타입이 결정되고 변수의 타입을 변경할 수 없다.
- 자바스크립트의 변수는 선언이 아닌 할당에 의해 타입이 결정(타입 추론)된다. 그리고 재할당에 의해 변수의 타입은 언제든지 동적으로 변할 수 있다. 이런 특징을 동적 타이핑이라고 한다. 자바스크립트는 동적 타입 언어이다. 따라서 변수에 할당되어 있는 값에 의해 변수의 타입이 동적으로 결정된다. 동적 타입 언어는 유연성은 높지만 신뢰성은 떨어진다.
