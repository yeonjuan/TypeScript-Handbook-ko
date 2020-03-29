# 일반 타입 (General Types)

## `Number`, `String`, `Boolean`, `Symbol` and `Object`

`Number`, `String`, `Boolean`, `Symbol`, `Object` 타입을 사용*하지 마세요*.
이 타입들은 JavaScript 코드에서 거의 사용되지 않는 비-원시형 박싱된 객체를 가르킵니다.

```ts
/* 잘못됨 */
function reverse(s: String): String;
```

`number`, `string`, `boolean`, `symbol` 타입을 사용 *하세요*.

```ts
/* 좋음 */
function reverse(s: string): string;
```

`Object` 대신에, [TypeScript 2.2 에 추가된](../release%20notes/TypeScript%202.2.md#object-type) 비-원시형 `object`타입을 사용*하세요*.

## 제네릭 (Generics)

타입 매개변수를 사용하지 않는 제네릭 타입을 사용*하지 마세요*. 더 자세한 내용은 [TypeScript FAQ 페이지](https://github.com/Microsoft/TypeScript/wiki/FAQ#why-doesnt-type-inference-work-on-this-interface-interface-foot---)에서 확인하세요.

<!-- TODO: More -->

# 콜백 타입 (Callback Types)

## 콜백의 반환 타입 (Return Types of Callbacks)

<!-- TODO: Reword; these examples make no sense in the context of a declaration file -->

사용하지 않는 콜백의 반환 값 타입에 `any`를 사용*하지 마세요*:

```ts
/* 잘못됨 */
function fn(x: () => any) {
    x();
}
```

사용하지 않는 콜백의 반환 값 타입은 `void`를 사용*하세요*:  

```ts
/* 좋음 */
function fn(x: () => void) {
    x();
}
```

*이유*: `void`를 사용하면 실수로 `x`의 반환 값을 사용하는 것을 방지 할 수 있기 때문에 더 안전합니다.:

```ts
function fn(x: () => void) {
    var k = x(); // oops! meant to do something else
    k.doSomething(); // error, but would be OK if the return type had been 'any'
}
```

## 콜백에서 선택적 매개변수 (Optional Parameters in Callbacks)

정말 의도한 것이 아니라면 콜백에 선택적 매개변수를 사용*하지 마세요*:

```ts
/* 잘못됨 */
interface Fetcher {
    getObject(done: (data: any, elapsedTime?: number) => void): void;
}
```

이는 아주 구체적인 의미를 가지고 있습니다: `done` 콜백은 1개 혹은 2개의 인자로 호출될 수 있습니다.
작성자는 아마 `elapsedTime` 매개변수가 콜백에 상관없다는 것을 말하려는 의도였을 것입니다,
  하지만 이를 위해 매개변수를 선택적으로 만들 필요는 없습니다 --
  콜백에 더 적은 인수를 제공하는 것은 항상 허용됩니다.

콜백 매개변수를 비-선택적으로 작성*하세요*:

```ts
/* 좋음 */
interface Fetcher {
    getObject(done: (data: any, elapsedTime: number) => void): void;
}
```

## 오버로드와 콜백 (Overloads and Callbacks)

콜백의 인수만 다른 오버로드를 분리해서 작성 *하지 마세요*:

```ts
/* 잘못됨 */
declare function beforeAll(action: () => void, timeout?: number): void;
declare function beforeAll(action: (done: DoneFn) => void, timeout?: number): void;
```

최대 인수를 사용해 하나의 오버로드를 작성 *하세요*:

```ts
/* 좋음 */
declare function beforeAll(action: (done: DoneFn) => void, timeout?: number): void;
```

*이유*: 콜백이 매개변수를 무시하는 것은 항상 허용되므로, 짧은 오버로드는 필요하지 않습니다.
더 짧은 콜백을 먼저 작성하면 넘어오는 함수가 첫 번째 오버로드와 일치하기 때문에 잘못된-타입의 함수를 허용합니다.

# 함수 오버로드 (Function Overloads)

## 순서 (Ordering)

더 일반적인 오버로드를 더 구체적인 오버로드 이전에 두지 *마세요*:

```ts
/* 잘못됨 */
declare function fn(x: any): any;
declare function fn(x: HTMLElement): number;
declare function fn(x: HTMLDivElement): string;

var myElem: HTMLDivElement;
var x = fn(myElem); // x: any, wat?
```

구체적인 오버로드 뒤에 일반적인 오버로드가 위치하게 정렬 *하세요*:

```ts
/* 좋음 */
declare function fn(x: HTMLDivElement): string;
declare function fn(x: HTMLElement): number;
declare function fn(x: any): any;

var myElem: HTMLDivElement;
var x = fn(myElem); // x: string, :)
```

*이유*: TypeScript는 함수 호출을 처리할 때 *첫 번째로 일치하는 오버로드*를 선택합니다.
이전의 오버로드가 뒤에 것보다 "더 구체적"이면, 뒤에 것은 사실상 가려져 호출되지 않습니다.

## Use Optional Parameters

*Don't* write several overloads that differ only in trailing parameters:

```ts
/* WRONG */
interface Example {
    diff(one: string): number;
    diff(one: string, two: string): number;
    diff(one: string, two: string, three: boolean): number;
}
```

*Do* use optional parameters whenever possible:

```ts
/* OK */
interface Example {
    diff(one: string, two?: string, three?: boolean): number;
}
```

Note that this collapsing should only occur when all overloads have the same return type.

*Why*: This is important for two reasons.

TypeScript resolves signature compatibility by seeing if any signature of the target can be invoked with the arguments of the source,
  *and extraneous arguments are allowed*.
This code, for example, exposes a bug only when the signature is correctly written using optional parameters:

```ts
function fn(x: (a: string, b: number, c: number) => void) { }
var x: Example;
// When written with overloads, OK -- used first overload
// When written with optionals, correctly an error
fn(x.diff);
```

The second reason is when a consumer uses the "strict null checking" feature of TypeScript.
Because unspecified parameters appear as `undefined` in JavaScript, it's usually fine to pass an explicit `undefined` to a function with optional arguments.
This code, for example, should be OK under strict nulls:

```ts
var x: Example;
// When written with overloads, incorrectly an error because of passing 'undefined' to 'string'
// When written with optionals, correctly OK
x.diff("something", true ? undefined : "hour");
```

## Use Union Types

*Don't* write overloads that differ by type in only one argument position:

```ts
/* WRONG */
interface Moment {
    utcOffset(): number;
    utcOffset(b: number): Moment;
    utcOffset(b: string): Moment;
}
```

*Do* use union types whenever possible:

```ts
/* OK */
interface Moment {
    utcOffset(): number;
    utcOffset(b: number|string): Moment;
}
```

Note that we didn't make `b` optional here because the return types of the signatures differ.

*Why*: This is important for people who are "passing through" a value to your function:

```ts
function fn(x: string): void;
function fn(x: number): void;
function fn(x: number|string) {
    // When written with separate overloads, incorrectly an error
    // When written with union types, correctly OK
    return moment().utcOffset(x);
}
```
