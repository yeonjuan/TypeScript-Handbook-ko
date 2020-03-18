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
/* WRONG */
function fn(x: () => any) {
    x();
}
```

사용하지 않는 콜백의 반환 값 타입은 `void`를 사용*하세요*:  

```ts
/* OK */
function fn(x: () => void) {
    x();
}
```

*왜*: `void`를 사용하면 실수로 `x`의 반환 값을 사용하는 것을 방지 할 수 있기 때문에 더 안전합니다.:

```ts
function fn(x: () => void) {
    var k = x(); // oops! meant to do something else
    k.doSomething(); // error, but would be OK if the return type had been 'any'
}
```

## 콜백에 선택적 매개변수 (Optional Parameters in Callbacks)

정말 의도한 것이 아니라면 콜백에 선택적 매개변수를 사용*하지 마세요*:

```ts
/* WRONG */
interface Fetcher {
    getObject(done: (data: any, elapsedTime?: number) => void): void;
}
```

This has a very specific meaning: the `done` callback might be invoked with 1 argument or might be invoked with 2 arguments.
The author probably intended to say that the callback might not care about the `elapsedTime` parameter,
  but there's no need to make the parameter optional to accomplish this --
  it's always legal to provide a callback that accepts fewer arguments.

*Do* write callback parameters as non-optional:

```ts
/* OK */
interface Fetcher {
    getObject(done: (data: any, elapsedTime: number) => void): void;
}
```

## Overloads and Callbacks

*Don't* write separate overloads that differ only on callback arity:

```ts
/* WRONG */
declare function beforeAll(action: () => void, timeout?: number): void;
declare function beforeAll(action: (done: DoneFn) => void, timeout?: number): void;
```

*Do* write a single overload using the maximum arity:

```ts
/* OK */
declare function beforeAll(action: (done: DoneFn) => void, timeout?: number): void;
```

*Why*: It's always legal for a callback to disregard a parameter, so there's no need for the shorter overload.
Providing a shorter callback first allows incorrectly-typed functions to be passed in because they match the first overload.

# Function Overloads

## Ordering

*Don't* put more general overloads before more specific overloads:

```ts
/* WRONG */
declare function fn(x: any): any;
declare function fn(x: HTMLElement): number;
declare function fn(x: HTMLDivElement): string;

var myElem: HTMLDivElement;
var x = fn(myElem); // x: any, wat?
```

*Do* sort overloads by putting the more general signatures after more specific signatures:

```ts
/* OK */
declare function fn(x: HTMLDivElement): string;
declare function fn(x: HTMLElement): number;
declare function fn(x: any): any;

var myElem: HTMLDivElement;
var x = fn(myElem); // x: string, :)
```

*Why*: TypeScript chooses the *first matching overload* when resolving function calls.
When an earlier overload is "more general" than a later one, the later one is effectively hidden and cannot be called.

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
