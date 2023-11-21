# 4장. 타입 설계

## 28. 유효한 상태만 표현하는 타입을 지향하기

```ts
// 😰
interface State {
  pageText: string;
  isLoading: boolean;
  error?: string;
}

function renderPage(state: State) {
  if (state.error) {
    return `Error! Unable to load ${currentPage}: ${state.error}`;
  } else if (state.isLoading) {
    return `Loading ${currentPage}...`;
  }
  return `<h1>${currentPage}</h1>\n${state.pageText}`;
}
// - 분기 조건이 명확히 분리되어 있지 않다
// - isLoading이 true이고 동시에 error값이 존재하면 로딩 중인 상태인지 오류가 발생한 상태인지 명확히 구분할 수 없다.

async function changePage(state: State, newPage: string) {
  state.isLoading = true;
  try {
    const response = await fetch(getUrlForPage(newPage));
    if (!response.ok) {
      throw new Error(`Unable to load ${newPage}: ${response.statusText}`);
    }
    const text = await response.text();
    state.isLoading = false;
    state.pageText = text;
  } catch (e) {
    state.error = "" + e;
  }
}
// - 오류가 발생했을 때 state.isLoading을 false로 설정하는 로직이 빠져있다.
// - state.error를 초기화하지 않았기 때문에, 페이지 전환 중에 로딩 메시지 대신 과거의 오류 메세지를 보여준다.
// - 페이지 로딩 중 사용자가 페이지를 전환하면 어떤 일이 벌어질지 예상하기 어렵다.

// 문제는 상태 값의 두 가지 속성이 동시에 정보가 부족하거나 두 가지 속성이 충돌할 수 있다

// 👍
interface RequestPending {
  state: "pending";
}
interface RequestError {
  state: "error";
  error: string;
}
interface RequestSuccess {
  state: "ok";
  pageText: string;
}

type RequestState = RequestPending | RequestError | RequestSuccess;

interface State {
  currentPage: string;
  requests: { [page: string]: RequestState };
}
// 각각의 상태를 명시적으로 모델링하는 태그된 유니온을 사용

function renderPage(state: State) {
  const { currentPage } = state;
  const requestState = state.requests[currentPage];
  switch (requestState.state) {
    case "pending":
      return `Loading ${currentPage}...`;
    case "error":
      return `Error! Unable to load ${currentPage}: ${state.error}`;
    case "ok":
      return `<h1>${currentPage}</h1>\n${state.pageText}`;
  }

  async function changePage(state: State, newPage: string) {
    state.requests[newPage] = { state: "pending" };
    state.currentPage = newPage;
    try {
      const response = await fetch(getUrlForPage(newPage));
      if (!response.ok) {
        throw new Error(`Unable to load ${newPage}: ${response.statusText}`);
      }
      const pageText = await response.text();
      state.requests[newPage] = { state: "ok", pageText };
    } catch (e) {
      state.requests[newPage] = { state: "error", error: "" + e };
    }
  }
}
```

## 29. 사용할 때는 너그럽게, 생성할 때는 엄격하게

- 함수의 매개변수는 타입의 범위가 넓어도 되지만, 결과를 반환할 때는 일반적으로 타입의 범위가 더 구체적이어야 한다.

```ts
// 😰
declare function setCamera(camera: CameraOptions): void;
declare function viewportForBounds(bounds: LngLatBounds): CameraOptions;

interface CameraOptions {
  center?: LngLat;
  zoom?: number;
  bearing?: number;
  pitch?: number;
}
type LngLat =
  | { lng: number; lat: number }
  | { lon: number; lat: number }
  | [number, number];
type LngLatBounds =
  | {
      northeast: LngLat;
      southwest: LngLat;
    }
  | [LngLat, LngLat]
  | [number, number, number, number];

function focusOnFeature(f: Feature) {
  const bounds = calculateBoundingBox(f);
  const camera = viewportForBounds(bounds);
  setCamera(camera);
  const {
    center: { lat, lng }, // ...형식에 'lat' 속성이 없습니다. ...형식에 'lng' 속성이 없습니다.
    zoom,
  } = camera;
  zoom; // 타입이 number | undefined
}

// 👍
interface LngLat {
  lng: number;
  lat: number;
}
type LngLatLike = LngLat | { lon: number; lat: number } | [number, number];

interface Camera {
  center: LngLat;
  zoom: number;
  bearing: number;
  pitch: number;
}
interface CameraOptions extends Omit<Partial<Camera>, "center"> {
  center?: LngLatLike;
}
// 또는
// interface CameraOptions {
//   center?: LngLatLike;
//   zoom?: number;
//   bearing?: number;
//   pitch?: number;
// }
type LngLatBounds =
  | {
      northeast: LngLat;
      southwest: LngLat;
    }
  | [LngLat, LngLat]
  | [number, number, number, number];

declare function setCamera(camera: CameraOptions): void;
declare function viewportForBounds(bounds: LngLatBounds): Camera;
```

## 30. 문서에 타입 정보를 쓰지 않기

- 주석과 변수명에 타입 정보를 적는 것은 피해야 한다.
- 타입이 명확하지 않은 경우 변수명에 단위 정보를 포함하는 것을 고려하는 것이 좋다. (timeMs 또는 temperatureC)

## 31. 타입 주변에 null값 배치하기

- 한 값의 null 여부가 다른 값의 null 여부에 암시적으로 관련되도록 설계하면 안된다.

```ts
// 😰
function extent(nums: number[]) {
  let min, max;
  for (const num of nums) {
    if (!min) {
      min = num;
      max = num;
    } else {
      min = Math.min(min, num); // 'number | undefined' 형식의 인수는 'number'형식의 매개변수에 할당될 수 없다.
      max = Math.max(min, num);
    }
  }
  return [min, max];
}

const [min, max] = extent([0, 1, 2]);
const span = max - min; // 개체가 'undefined'인 것 같습니다.

// 👍
function extent(nums: number[]) {
  let result: [number, number] | null = null;
  for (const num of nums) {
    if (!result) {
      result = [num, num];
    } else {
      result = [Math.min(num, result[0]), Math.max(num, result[1])];
    }
  }
  return [min, max];
}

// 또는 null아님 단언(!)을 사용하면 min과 max를 얻을 수 있다.
const [min, max] = extent([0, 1, 2])!;
const span = max - min; // 정상

// 또는 단순 if구문으로 체크할 수 있다.
const range = extent([0, 1, 2]);
if (range) {
  const [min, max] = range;
  const span = max - min; // 정상
}
```

## 32. 유니온의 인터페이스보다는 인터페이스의 유니온을 사용하기

- 유니온 타입의 속성을 여러 개 가지는 인터페이스에서는 속성 간의 관계가 분명하지 않기 때문에 실수가 자주 발생하므로 주의해야 한다.
- 유니온의 인터페이스보다 인터페이스의 유니온이 더 정확하고 타입스크립트가 이해하기도 좋다.
- 타입스크립트가 제어 흐름을 분석할 수 있도록 타입에 태그를 넣는 것을 고려해야 한다.

```ts
// 😰
interface Layer {
  layout: FillLayout | LineLayout | PointLayout; // 모양이 그려지는 방법과 위치(둥근 모서리, 직선)
  paint: FillPaint | LinePaint | PointPaint; // 스타일(파란선, 굵은선, 얇은선, 점선)
} // layout이 LineLayout이면서 paint 속성이 FillPaint 타입인 것은 말이 안된다.

// 👍
interface FillLayer {
  layout: FillLayout;
  paint: FillPaint;
}
interface LineLayer {
  layout: LineLayout;
  paint: LinePaint;
}
interface PointLayer {
  layout: PointLayout;
  paint: PointPaint;
}
type Layer = FillLayer | LineLayer | PointLayer;

// 😰
interface Layer {
  type: "fill" | "line" | "point";
  layout: FillLayout | LineLayout | PointLayout;
  paint: FillPaint | LinePaint | PointPaint;
}

// 👍
// type 속성은 '태그'이며 런타임에 어떤 타입의 Layer가 사용되는지 판단하는데 쓰인다.
interface FillLayer {
  type: "fill";
  layout: FillLayout;
  paint: FillPaint;
}
interface LineLayer {
  type: "line";
  layout: LineLayout;
  paint: LinePaint;
}
interface PointLayer {
  type: "point";
  layout: PointLayout;
  paint: PointPaint;
}
type Layer = FillLayer | LineLayer | PointLayer;

function drawLayer(layer: Layer) {
  if (layer.type === "fill") {
    const { paint } = layer; // 타입이 FillPaint
    const { layout } = layer; // 타입이 FillLayout
  } else if (layer.type === "line") {
    // ..
  }
}
```

```ts
// 😰
interface Person {
  name: string;
  // 다음은 둘 다 동시에 있거나 동시에 없다.
  placeOfBirth?: string;
  dateOfBirth?: Date;
}

// 👍
// 두 개의 속성을 하나의 객체로 모으는 것이 좋은 설계이다.
interface Person {
  name: string;
  birth?: {
    place: string;
    date: Date;
  };
}
```

## 33. string 타입보다 더 구체적인 타입 사용하기

- 타입을 명시적으로 정의함으로써 다른 곳으로 값이 전달되어도 타입 정보가 유지된다.
- 해당 타입의 의미를 설명하는 주석을 붙여 놓을 수 있다.

```ts
// 😰
interface Album {
  artist: string;
  title: string;
  releaseDate: string; // YYYY-MM-DD
  recordingType: string; // 'live' 또는 'studio'
}

// 👍
/** 이 녹음은 어떤 환경에서 이루어졌는지? */
type RecordingType = "studio" | "live";
interface Album {
  artist: string;
  title: string;
  releaseDate: Date;
  recordingType: RecordingType;
}
```

- `keyof` 연산자로 더욱 세밀하게 객체의 속성 체크가 가능해진다.

```ts
// 어떤 배열에서 한 필드의 값만 추출하는 함수
// 😰
function pluck(records: any[], key: string): any[] {
  return records.map((r) => r[key]);
}

// type K = keyof Album;
// 타입이 "artist" | "title" | "releaseDate" | "recordingType"
function pluck<T>(records: T[], key: keyof T) {
  return records.map((r) => r[key]);
}
// 추론된 타입은 function pluck<T>(records: T[], key: keyof T): T[keyof T][]

const releaseDates = pluck(albums, "releaseDate"); // 타입이 (string | Date)[]
// 올바른 releaseDates의 타입은 Date[]이어야 한다.

// 👍
function pluck<T, K extends keyof T>(records: T[], key: K): T[K][] {
  return records.map((r) => r[key]);
}
```

## 34. 부정확한 타입보다는 미완성 타입을 사용하기

- 타입이 없는 것보다 잘못된 게 더 나쁘다.
- 정확하게 타입을 모델링할 수 없다면 부정확하게 모델링하지 말아야 한다. 또는 `any`와 `unknown`를 구별해서 사용해야 한다.

## 35. 데이터가 아닌 API와 명세를 보고 타입 만들기

- 타입의 안전성을 얻기 위해 API 또는 데이터 형식에 대한 타입 생성을 고려해야 한다.
- 데이터에 드러나지 않는 예외적인 경우들이 문제가 될 수 있기 때문에 데이터보다는 명세로부터 코드를 생성하는 것이 좋다.

## 36. 해당 분야의 용어로 타입 이름 짓기

- 가독성을 높이고, 추상화 수주을 올리기 위해서 해당 분야의 용어를 사용해야 한다.
- data, info, thing, item... 같은 모호하고 의미 없는 이름은 피해야 한다.

## 37. 공식 명칭에는 상표를 붙이기

- 타입스크립트는 구조적 타이핑(덕 타이핑)을 사용하기 때문에 값을 세밀하게 구분하지 못하는 경우가 있다. 값을 구분하기 위해 공식 명칭이 필요하다면 상표를 붙이는 것을 고려해야 한다.
- 상표 기법은 타입 시스템에서 동작하지만 런타임에 상표를 검사하는 것과 동일한 효과를 얻을 수 있다

```ts
interface Vector2D {
  x: number;
  y: number;
}

function calculateNorm(p: Vector2D) {
  return Math.sqrt(p.x * p.x + p.y * p.y);
}

calculateNorm({ x: 3, y: 4 }); // 정상, 결과는 5
const vec3D = { x: 3, y: 4, z: 1 };
calculateNorm(vec3D); // 정상, 결과는 5
```

- 구조적 타이핑 관점에서 문제가 없기는 하지만 수학적으로 따지면 2차원 벡터를 사용해야 이치에 맞다.
- calculateNorm 함수가 3차원 벡터를 허용하지 않게 하려면 공식 명칭을 사용하면 된다. ('상표' 붙이기)

```ts
interface Vector2D {
  _brand: "2d";
  x: number;
  y: number;
}

function vec2D(x: number, y: number): Vector2D {
  return { x, y, _brand: "2d" };
}
function calculateNorm(p: Vector2D) {
  return Math.sqrt(p.x * p.x + p.y * p.y);
}

calculateNorm(vec2D(3, 4)); // 정상, 결과는 5
const vec3D = { x: 3, y: 4, z: 1 };
calculateNorm(vec3D); // '_brand' 속성이 ... 형식에 없습니다.
```

```ts
type AbsolutePath = string & { _brand: "abs" };
function listAbsolutePath(path: AbsolutePath) {
  // ...
}
function isAbsolutePath(path: string): path is AbsolutePath {
  return path.startsWith("/");
}

// path 값이 절대경로와 상대경로 둘 다 될 수 있다면, 타입을 정제해 주는 타입 가드를 사용해서 오류를 방지할 수 있다.
function f(path: string) {
  if (isAbsolutePath(path)) {
    listAbsolutePath(path);
  }
  listAbsolutePath(path);
}
```
