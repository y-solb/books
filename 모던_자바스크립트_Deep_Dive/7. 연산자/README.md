# 7장. 연산자

## 7.1 산술 연산자

- 피연산자 앞에 전위 증가/감소 연산자는 먼저 피연산자의 값을 증가/감소시킨 후, 다른 연산을 수행한다.
- 피연산자 뒤에 위치한 후위 증가/감소 연산자는 먼저 다른 연산을 수행한 후, 피연산자의 값을 증가/감소시킨다.

```js
var x = 5,
  result;

// 선할당 후증가
result = x++;
console.log(result, x); // 5 6

// 선증가 후할당
result = ++x;
console.log(result, x); // 7 7
```

## 7.3 비교 연산자

- 동등 비교 연산자는 좌항과 우항의 피연산자를 비교할 때 먼저 암묵적 타입 변환을 통해 타입을 일치시킨 후 같은 값인지 비교한다.
- 일치 비교 연산자는 좌항과 우항의 피연산자가 타입도 같고 값도 같은 경우에 한하여 `true`를 반환한다.
