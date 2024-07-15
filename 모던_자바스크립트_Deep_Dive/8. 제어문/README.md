# 8장. 제어문

## 8.3 반복문

### 8.3.2 while문

for문은 반복 횟수가 명확할 때 주로 사용하고 while문은 반복 횟수가 불명확할 때 주로 사용한다.

```js
var count = 0;

while (count < 3) {
  console.log(count); // 0 1 2
  count++;
}
```

### 8.3.3 do ... while문

코드 블록을 먼저 실행하고 조건식을 평가한다. 무조건 한번 이상 실행된다.

```js
var count = 0;

do (true) {
console.log(count); // 0 1 2
count++;
} while (count < 3)
```

## 8.4 break문

레이블 문, 반복문 또는 switch 문의 코드 블록을 탈출한다.

```js
var string = "Hello World.";
var search = "l";
var index;

for (var i = 0; i < string.length; i++) {
  if (string[i] === search) {
    index = i;
    break; // 반복문 탈출
  }
}
```

## 8.5 continue문

반복문의 코드 블록을 현 지점에서 중단하고 반복문의 증감식으로 실행 흐름을 이동시킨다. break문처럼 반복문을 탈출하지는 않는다.

```js
var string = "Hello World.";
var search = "l";
var count = 0;

for (var i = 0; i < string.length; i++) {
  // 'l'이 아니면 현 지점에서 실행을 중단하고 반복문의 증감식으로 이동한다.
  if (string[i] !== search) continue; // 빠른 return!
  count++; // continue문이 실행되면 이 문은 실핼되지 않는다.
}
```
